# my-vllm

LLM
**init**:
  创建多进程
  创建ModelRunner
    **init**：
      初始化分布式
      创建模型
      定义采样器Sampler
      预热模型
        run：
          使用全为0的空序列进行计算，这段序列默认是prefill阶段
          填充到context中
          返回inputs和position ids
          在使用prefill版本的时候不使用flash attention
      分配kv cache：
        检查显存最大峰值
      捕获cuda图：
        将计算图信息全都捕获下来，从而实现“完全算子融合”
        捕获的计算图只能接受固定大小的输入
      继续设置在cpu指定，为了防止后续操作再gpu上再执行无关操作
      分配共享内存
      
  创建Scheduler：
    创建BlockManager
      **init**：
        创建列表，列表里放入Block
        创建使用列表
        创建空列表
    创建两个队列，分别负责wating和running

**generate**:
  add_request:
    创建序列
      Sequence：
        默认是waiting的在开始
        存储block id
        最新的token
    将序列推入scheduler的waiting队列
    获取一个输入：
      先处理prefill，不可以同时处理prefilling和decoding
        batch不能超过，同时不能超过所有的block使用量
        使用scheduler给这个序列alloc：
          循环这个序列的所有block进行处理
      再处理decoding
        逐步读取running队列的数据，如果running数据已经满了，采取末尾淘汰制
      将每条数据和参数设置好
  执行generate：
    对于prefill数据，不使用计算图
    对于decode数据，使用计算图
