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
        将计算图信息全都捕获下来，从而实现完全算子融合
      继续设置在cpu指定，为了防止后续操作再gpu上再执行无关操作
      分配共享内存
      
  创建Scheduler(config)
