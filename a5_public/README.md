# CS224n Assignment 5
Public version of assignment 5 from CS224n Winter 2020 coures website, [`a5_publc.zip`](http://web.stanford.edu/class/cs224n/assignments/a5_public.zip). For some reason the assignment handout is the same across both versions. Not sure what's the difference.

- 2020/02/16 - This network is much harder to train despite of consistently having around 1200 words/second on average vs 700 words/second on average in assignment 4. By 15 hours mark, its only at epoch 15 with loss hovering around 90 using RTX 2080 Ti. VSCode screwed up and exited at around 20 hours mark at epoch 19 with loss hovering around 80-90, test BLEU score 24.35.

- 2020/02/17 - Restarted run, using `batch_size=64` vs default 32, and set `max_epoch=60` vs default 30. GPU memory at 10/11GB. Training reached max epoch after around 34 hours, with training loss at the low 70s and validation perplexity at 59. Average words per second is around 2000 words per second.

## Unit tests:
Test cases for modules are included in files prefix `test`. Run them by:

```python test_highway.py```


```python test_cnn.py```


## Notes:
- Changed padding token to `<pad>` from `'∏'` in `vocab.py` and `sanity_check.py` to match `char_vocab_sanity_check.json`.
- Changed padding token to `<unk>` from `'Û'` in `vocab.py` and `sanity_check.py` to match `char_vocab_sanity_check.json`.
- Modification of provided code in `nmt_model.py`:
```
###########
# 2020/02/15 This line throws an runtime error:
# RuntimeError: view size is not compatible with input tensor's size and stride 
# (at least one dimension spans across two contiguous subspaces). 
# Use .reshape(...) instead.
# target_chars = target_padded_chars[1:].view(-1, max_word_len)
target_chars = target_padded_chars[1:].reshape(-1, max_word_len)
###########
```
- Official handout `a5.pdf` does not state clearly (or reminds students) to copy functions from `assignment 4`. It's pretty easy to miss one or a few of them. The following are copied from `assignment 4`:
    - `pad_sents` in `utils.py`
    - `__init__` in `nmt_model.py`
    - `encode` in `nmt_model.py`
    - `decode` in `nmt_model.py`
    - `step` in `nmt_model.py`

- Seemingly compatbility problem. Without using `ceil_mode=True`, `sh run.sh train_local_q1` will fail to run. After changing `ceil_mode=True` in `cnn.py`, our network can pass `--no-char-decoder` with <1 loss and >99 BLEU score. 2020/02/16: switched to using `torch.max` since `ceil_mode=True` might returns dimension > 1, causing squeeze to not squeeze and dimension mismatch.
```
#############################
# Setting ceil_mode=True, otherwise it throws
# RuntimeError: Given input size: (256x1x12). Calculated output size: (256x1x0). Output size is too small
# References:
# https://github.com/pytorch/pytorch/issues/28625#issue-512206689
# https://github.com/pytorch/pytorch/issues/26610#issue-496710180
#############################
self.maxpool = nn.MaxPool1d(kernel_size=max_len - kernel_size + 1, ceil_mode=True)
```
- Problem 2(d) trains fine. But when running tests it throws the error. 2020/02/16: Fixed. The reason is when using ceil_mode=True, maxpool might return a dimension of 2 instead of 1, causing dimension mismatch. Switched to using torch.max.
- Modification of provided code in `nmt_model.py`:
```
###########
# 2020/02/15 This line throws an runtime error:
# RuntimeError: view size is not compatible with input tensor's size and stride 
# (at least one dimension spans across two contiguous subspaces). 
# Use .reshape(...) instead.
# target_chars = target_padded_chars[1:].view(-1, max_word_len)
target_chars = target_padded_chars[1:].reshape(-1, max_word_len)
###########
```            
- Problem 2(d) trains fine. But when running tests it throws the error.
```
epoch 200, iter 1000, cum. loss 0.18, cum. ppl 1.01 cum. examples 200   
begin validation ...
validation: iter 1000, dev. ppl 1.002426
save currently the best model to [model.bin]
save model parameters to [model.bin]
```
```
(pytorch) PS B:\user\cs224n\a5_public> python run.py decode model.bin ./en_es_data/test_tiny.es ./en_es_data/test_tiny.en outputs/test_outputs_local_q2.txt
[nltk_data] Downloading package punkt to
[nltk_data]     C:\Users\HCL\AppData\Roaming\nltk_data...
[nltk_data]   Package punkt is already up-to-date!
load test source sentences from [./en_es_data/test_tiny.es]
load test target sentences from [./en_es_data/test_tiny.en]
load model from model.bin
Decoding:   0%|                                  | 0/4 [00:00<?, ?it/s] 
  File "run.py", line 350, in <module>
    main()
  File "run.py", line 344, in main
    decode(args)
  File "run.py", line 286, in decode
    max_decoding_time_step=int(args['--max-decoding-time-step']))       
  File "run.py", line 317, in beam_search
    example_hyps = model.beam_search(src_sent, beam_size=beam_size, max_decoding_time_step=max_decoding_time_step)
  File "B:\user\cs224n\a5_public\nmt_model.py", line 358, in beam_search
    y_t_embed = self.model_embeddings_target(y_tm1)
  File "B:\miniconda\envs\pytorch\lib\site-packages\torch\nn\modules\module.py", line 532, in __call__
    result = self.forward(*input, **kwargs)
  File "B:\user\cs224n\a5_public\model_embeddings.py", line 89, in forward
    X_highway = self.highway(X_conv_out)
  File "B:\miniconda\envs\pytorch\lib\site-packages\torch\nn\modules\module.py", line 532, in __call__
    result = self.forward(*input, **kwargs)
  File "B:\user\cs224n\a5_public\highway.py", line 41, in forward       
    X_proj = F.relu(self.proj(X_conv_out))
  File "B:\miniconda\envs\pytorch\lib\site-packages\torch\nn\modules\module.py", line 532, in __call__
    result = self.forward(*input, **kwargs)
  File "B:\miniconda\envs\pytorch\lib\site-packages\torch\nn\modules\linear.py", line 87, in forward
    return F.linear(input, self.weight, self.bias)
  File "B:\miniconda\envs\pytorch\lib\site-packages\torch\nn\functional.py", line 1372, in linear
    output = input.matmul(weight.t())
RuntimeError: size mismatch, m1: [1280 x 2], m2: [256 x 256] at C:\w\1\s\tmp_conda_3.6_095855\conda\conda-bld\pytorch_1579082406639\work\aten\src\TH/generic/THTensorMath.cpp:136
```