> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [efficientdl.com](https://efficientdl.com/faster-deep-learning-in-pytorch-a-guide/)

> An overview of some of the lowest-effort, highest-impact ways of accelerating the training of deep le......

Say, you're training a deep learning model in PyTorch. What can you do to make your training finish faster?

In this post, I'll provide an overview of some of the lowest-effort, highest-impact ways of accelerating the training of deep learning models in PyTorch. For each of the methods, I'll briefly summarize the idea, try to estimate the expected speed-up and discuss some limitations. I will focus on conveying the most important parts and point to further resources for each of them. Mostly, I'll focus on changes that can be made directly within PyTorch without introducing additional libraries and I'll assume that you are training your model on GPU(s).

The suggestions – roughly sorted from largest to smallest expected speed-up – are:

1.  [Consider using a different learning rate schedule.](#1-consider-using-another-learning-rate-schedule)
2.  [Use multiple workers and pinned memory in `DataLoader`.](#2-use-multiple-workers-and-pinned-memory-in-dataloader)
3.  [Max out the batch size.](#3-max-out-the-batch-size)
4.  [Use Automatic Mixed Precision (AMP).](#4-use-automatic-mixed-precision-amp-)
5.  [Consider using a different optimizer.](#5-consider-using-another-optimizer)
6.  [Turn on cudNN benchmarking.](#6-turn-on-cudnn-benchmarking)
7.  [Beware of frequently transferring data between CPUs and GPUs.](#7-beware-of-frequently-transferring-data-between-cpus-and-gpus)
8.  [Use gradient/activation checkpointing.](#8-use-gradient-activation-checkpointing)
9.  [Use gradient accumulation.](#9-use-gradient-accumulation)
10.  [Use DistributedDataParallel for multi-GPU training.](#10-use-distributed-data-parallel-for-multi-gpu-training)
11.  [Set gradients to None rather than 0.](#11-set-gradients-to-none-rather-than-0)
12.  [Use `.as_tensor` rather than `.tensor()`](#12-use-as_tensor-rather-than-tensor-)
13.  [Turn off debugging APIs if not needed.](#13-turn-on-debugging-tools-only-when-actually-needed)
14.  [Use gradient clipping.](#14-use-gradient-clipping)
15.  [Turn off bias before BatchNorm.](#15-turn-off-bias-before-batchnorm)
16.  [Turn off gradient computation during validation.](#16-turn-off-gradient-computation-during-validation)
17.  [Use input and batch normalization.](#17-use-input-and-batch-normalization)

### 1. Consider using another learning rate schedule

The learning rate (schedule) you choose has a large impact on the speed of convergence as well as the generalization performance of your model.

Cyclical Learning Rates and the 1Cycle learning rate schedule are both methods introduced by Leslie N. Smith ([here](https://arxiv.org/pdf/1506.01186.pdf) and [here](https://arxiv.org/abs/1708.07120)), and then popularised by fast.ai's Jeremy Howard and Sylvain Gugger ([here](https://www.fast.ai/2018/07/02/adam-weight-decay/) and [here](https://github.com/sgugger/Deep-Learning/blob/master/Cyclical%20LR%20and%20momentums.ipynb)). Essentially, the 1Cycle learning rate schedule looks something like this:

![](https://efficientdl.com/content/images/2021/05/image.png)[Source](https://sgugger.github.io/images/art5_lr_schedule.png)

Sylvain writes:

> [1cycle consists of]  two steps of equal lengths, one going from a lower learning rate to a higher one than go back to the minimum. The maximum should be the value picked with the Learning Rate Finder, and the lower one can be ten times lower. Then, the length of this cycle should be slightly less than the total number of epochs, and, in the last part of training, we should allow the learning rate to decrease more than the minimum, by several orders of magnitude.

In the best case this schedule achieves a massive speed-up – what Smith calls _Superconvergence_ – as compared to conventional learning rate schedules. Using the 1Cycle policy he needs ~10x fewer training iterations of a ResNet-56 on ImageNet to match the performance of the original paper, for instance). The schedule seems to perform robustly well across common architectures and optimizers.

PyTorch implements both of these methods `torch.optim.lr_scheduler.CyclicLR` and `torch.optim.lr_scheduler.OneCycleLR`, see [the documentation](https://pytorch.org/docs/stable/optim.html).

One drawback of these schedulers is that they introduce a number of additional hyperparameters. [This post](https://towardsdatascience.com/hyper-parameter-tuning-techniques-in-deep-learning-4dad592c63c8) and [this repo](https://github.com/davidtvs/pytorch-lr-finder), offer a nice overview and implementation of how good hyper-parameters can be found including the Learning Rate Finder mentioned above.

Why does this work? It doesn't seem entirely clear but one [possible explanation](https://arxiv.org/pdf/1506.01186.pdf) might be that regularly increasing the learning rate helps to traverse [saddle points in the loss landscape](https://papers.nips.cc/paper/2015/file/430c3626b879b4005d41b8a46172e0c0-Paper.pdf) more quickly.

### 2. Use multiple workers and pinned memory in `DataLoader`

When using [`torch.utils.data.DataLoader`](https://pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader), set `num_workers > 0`, rather than the default value of 0, and `pin_memory=True`, rather than the default value of `False`. Details of this are [explained here](https://pytorch.org/docs/stable/data.html).

[Szymon Micacz](https://nvlabs.github.io/eccv2020-mixed-precision-tutorial/files/szymon_migacz-pytorch-performance-tuning-guide.pdf) achieves a 2x speed-up for a single training epoch by using four workers and pinned memory.

A rule of thumb that [people are using](https://discuss.pytorch.org/t/guidelines-for-assigning-num-workers-to-dataloader/813/5) to choose the number of workers is to set it to four times the number of available GPUs with both a larger and smaller number of workers leading to a slow down.

Note that increasing `num_workers` will increase your CPU memory consumption.

### 3. Max out the batch size

This is a somewhat contentious point. Generally, however, it seems like using the largest batch size your GPU memory permits will accelerate your training (see [NVIDIA's Szymon Migacz](https://nvlabs.github.io/eccv2020-mixed-precision-tutorial/files/szymon_migacz-pytorch-performance-tuning-guide.pdf), for instance). Note that you will also have to adjust other hyperparameters, such as the learning rate, if you modify the batch size. A rule of thumb here is to double the learning rate as you double the batch size.

[OpenAI has a nice empirical paper](https://arxiv.org/pdf/1812.06162.pdf) on the number of convergence steps needed for different batch sizes. [Daniel Huynh](https://towardsdatascience.com/implementing-a-batch-size-finder-in-fastai-how-to-get-a-4x-speedup-with-better-generalization-813d686f6bdf) runs some experiments with different batch sizes (also using the 1Cycle policy discussed above) where he achieves a 4x speed-up by going from batch size 64 to 512.

[One of the downsides](https://arxiv.org/pdf/1609.04836.pdf) of using large batch sizes, however, is that they might lead to solutions that generalize worse than those trained with smaller batches.

### 4. Use Automatic Mixed Precision (AMP)

The release of PyTorch 1.6 included a native implementation of Automatic Mixed Precision training to PyTorch. The main idea here is that certain operations can be run faster and without a loss of accuracy at semi-precision (FP16) rather than in the single-precision (FP32) used elsewhere. AMP, then, automatically decide which operation should be executed in which format. This allows both for faster training and a smaller memory footprint.

In the best case, the usage of AMP would look something like this:

```
import torch

scaler = torch.cuda.amp.GradScaler()

for data, label in data_iter:
   optimizer.zero_grad()
   
   with torch.cuda.amp.autocast():
      loss = model(data)

   
   
   scaler.scale(loss).backward()

   
   
   scaler.step(optimizer)

   
   scaler.update()

```

[Source](https://pytorch.org/docs/stable/amp.html#gradient-scaling)

Benchmarking a number of common language and vision models on NVIDIA V100 GPUs, [Huang and colleagues find](https://pytorch.org/blog/accelerating-training-on-nvidia-gpus-with-pytorch-automatic-mixed-precision/) that using AMP over regular FP32 training yields roughly 2x – but upto 5.5x – training speed-ups.

Currently, only CUDA ops can be autocast in this way. See the [documentation](https://pytorch.org/docs/stable/amp.html#op-eligibility) here for more details on this and other limitations.

### 5. Consider using another optimizer

AdamW is Adam with weight decay (rather than L2-regularization) which was popularized by fast.ai and is now available natively in PyTorch as  
`torch.optim.AdamW`. AdamW seems to consistently outperform Adam in terms of both the error achieved and the training time. See [this excellent blog](https://www.fast.ai/2018/07/02/adam-weight-decay/) post on why using weight decay instead of L2-regularization makes a difference for Adam.

Both Adam and AdamW work well with the 1Cycle policy described above.

There are also a few not-yet-native optimizers that have received a lot of attention recently, most notably LARS ([pip installable implementation](https://github.com/kakaobrain/torchlars)) and [LAMB](https://github.com/cybertronai/pytorch-lamb).

NVIDA's APEX implements fused versions of a number of common optimizers such as [Adam](https://nvidia.github.io/apex/optimizers.html). This implementation avoid a number of passes to and from GPU memory as compared to the PyTorch implementation of Adam, yielding speed-ups in the range of 5%.

### 6. Turn on cudNN benchmarking

If your model architecture remains fixed and your input size stays constant, setting `torch.backends.cudnn.benchmark = True` might be beneficial ([docs](https://pytorch.org/docs/stable/backends.html#torch-backends-cudnn)). This enables the cudNN autotuner which will benchmark a number of different ways of computing convolutions in cudNN and then use the fastest method from then on.

For a rough reference on the type of speed-up you can expect from this, [Szymon Migacz](https://nvlabs.github.io/eccv2020-mixed-precision-tutorial/files/szymon_migacz-pytorch-performance-tuning-guide.pdf) achieves a speed-up of 70% on a forward pass for a convolution and a 27% speed-up for a forward + backward pass of the same convolution.

One caveat here is that this autotuning might become very slow if you max out the batch size as mentioned above.

### 7. Beware of frequently transferring data between CPUs and GPUs

Beware of frequently transferring tensors from a GPU to a CPU using`tensor.cpu()` and vice versa using `tensor.cuda()` as these are relatively expensive. The same applies for `.item()` and `.numpy()` – use `.detach()` instead.

If you are creating a new tensor, you can also directly assign it to your GPU using the keyword argument `device=torch****.****device('cuda:0')`.

If you do need to transfer data, using `.to(non_blocking=True)`, might be useful [as long as you don't have any synchronization points](https://discuss.pytorch.org/t/should-we-set-non-blocking-to-true/38234/4) after the transfer.

If you really have to, you might want to give Santosh Gupta's [SpeedTorch](https://github.com/Santosh-Gupta/SpeedTorch) a try, although it doesn't seem entirely clear when this actually does/doesn't provide speed-ups.

### 8. Use gradient/activation checkpointing

Quoting directly from the [documentation](https://pytorch.org/docs/stable/checkpoint.html):

> Checkpointing works by trading compute for memory. Rather than storing all intermediate activations of the entire computation graph for computing backward, the checkpointed part does **not** save intermediate activations, and instead recomputes them in backward pass. It can be applied on any part of a model.

> Specifically, in the forward pass, `function` will run in [`torch.no_grad()`](https://pytorch.org/docs/stable/generated/torch.no_grad.html#torch.no_grad) manner, i.e., not storing the intermediate activations. Instead, the forward pass saves the inputs tuple and the `function` parameter. In the backwards pass, the saved inputs and `function` is retrieved, and the forward pass is computed on `function` again, now tracking the intermediate activations, and then the gradients are calculated using these activation values.

So while this will might slightly increase your run time for a given batch size, you'll significantly reduce your memory footprint. This in turn will allow you to further increase the batch size you're using allowing for better GPU utilization.

While checkpointing is implemented natively as `torch.utils.checkpoint` ([docs](https://pytorch.org/docs/stable/checkpoint.html)), it does seem to take some thought and effort to implement properly. Priya Goyal [has a good tutorial](https://github.com/prigoyal/pytorch_memonger/blob/master/tutorial/Checkpointing_for_PyTorch_models.ipynb) demonstrating some of the key aspects of checkpointing.

### 9. Use gradient accumulation

Another approach to increasing the batch size is to accumulate gradients across multiple `.backward()` passes before calling `optimizer.step()`.

Following [a post](https://medium.com/huggingface/training-larger-batches-practical-tips-on-1-gpu-multi-gpu-distributed-setups-ec88c3e51255) by Hugging Face's Thomas Wolf, gradient accumulation can be implemented as follows:

```
@torch.jit.script
def fused_gelu(x):
    return x * 0.5 * (1.0 + torch.erf(x / 1.41421))


```

In this case, fusing the operations leads to a 5x speed-up for the execution of `fused_gelu` as compared to the unfused version.

See also [this post](https://pytorch.org/blog/optimizing-cuda-rnn-with-torchscript/) for an example of how Torchscript can be used to accelerate an RNN.

Hat tip to [u/Patient_Atmosphere45](https://www.reddit.com/user/Patient_Atmosphere45/) on Reddit for the suggestion.

### Sources and additional resources

Many of the tips listed above come from Szymon Migacz' [talk](https://www.youtube.com/watch?v=9mS1fIYj1So) and post in the [PyTorch docs](https://pytorch.org/tutorials/recipes/recipes/tuning_guide.html).

PyTorch Lightning's William Falcon has [two](https://towardsdatascience.com/9-tips-for-training-lightning-fast-neural-networks-in-pytorch-8e63a502f565) [interesting](https://towardsdatascience.com/7-tips-for-squeezing-maximum-performance-from-pytorch-ca4a40951259) posts with tips to speed-up training. [PyTorch Lightning](https://github.com/PyTorchLightning/pytorch-lightning) does already take care of some of the points above per-default.

Thomas Wolf at Hugging Face has a [number](https://medium.com/@Thomwolf) of interesting articles on accelerating deep learning – with a particular focus on language models.

The same goes for [Sylvain Gugger](https://sgugger.github.io/category/basics.html) and [Jeremy Howard](https://www.youtube.com/watch?v=LqGTFqPEXWs): they have many interesting posts in particular on [learning](https://sgugger.github.io/the-1cycle-policy.html) [rates](https://sgugger.github.io/how-do-you-find-a-good-learning-rate.html) and [AdamW](https://www.fast.ai/2018/07/02/adam-weight-decay/).

_Thanks to Ben Hahn, Kevin Klein and Robin Vaaler for their feedback on a draft of this post!_