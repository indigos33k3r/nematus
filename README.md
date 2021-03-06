NEMATUS
-------

Attention-based encoder-decoder model for neural machine translation

This package is based on the dl4mt-tutorial by Kyunghyun Cho et al. ( https://github.com/nyu-dl/dl4mt-tutorial ).
It was used to produce top-scoring systems at the WMT 16 shared translation task.

The changes to Nematus include:

  - the model has been re-implemented in tensorflow.
    See https://github.com/EdinburghNLP/nematus/tree/theano for the Theano-based version of Nematus.

  - new architecture variants for better performance:
     - arbitrary input features (factored neural machine translation) http://www.statmt.org/wmt16/pdf/W16-2209.pdf
     - deep models (Miceli Barone et al., 2017) https://arxiv.org/abs/1707.07631
     - dropout on all layers (Gal, 2015) http://arxiv.org/abs/1512.05287
     - tied embeddings (Press and Wolf, 2016) https://arxiv.org/abs/1608.05859
     - layer normalisation (Ba et al, 2016) https://arxiv.org/abs/1607.06450

 - improvements to scoring and decoding:
     - n-best output for decoder
     - scripts for scoring (given parallel corpus) and rescoring (of n-best output)

 - usability improvements:
     - command line interface for training
     - vocabulary files and model parameters are stored in JSON format (backward-compatible loading)
     - server mode

see changelog for more info.


SUPPORT
-------

For general support requests, there is a Google Groups mailing list at https://groups.google.com/d/forum/nematus-support . You can also send an e-mail to nematus-support@googlegroups.com .


INSTALLATION
------------

Nematus requires the following packages:

 - Python >= 2.7
 - tensorflow

To install tensorflow, we recommend following the steps at:
  ( https://www.tensorflow.org/install/ )

the following packages are optional, but *highly* recommended

 - CUDA >= 7  (only GPU training is sufficiently fast)
 - cuDNN >= 4 (speeds up training substantially)


DOCKER USAGE
------------

You can also create docker image by running following command, where you change `suffix` to either `cpu` or `gpu`:

`docker build -t nematus-docker -f Dockerfile.suffix .`

To run a CPU docker instance with the current working directory shared with the Docker container, execute:

``docker run -v `pwd`:/playground -it nematus-docker``

For GPU you need to have nvidia-docker installed and run:

``nvidia-docker run -v `pwd`:/playground -it nematus-docker``


TRAINING SPEED
--------------

Training speed depends heavily on having appropriate hardware (ideally a recent NVIDIA GPU),
and having installed the appropriate software packages.

To test your setup, we provide some speed benchmarks with `test/test_train.sh',
on an Intel Xeon CPU E5-2620 v4, with a Nvidia GeForce GTX Titan X (Pascal) and CUDA 9.0:


GPU, CuDNN 5.1, tensorflow 1.0.1:

  CUDA_VISIBLE_DEVICES=0 ./test_train.sh

>> 225.25 sentenses/s

 
USAGE INSTRUCTIONS
------------------

All of the scripts below can be run with `--help` flag to get usage information.

Sample commands with toy examples are available in the `test` directory;
for training a full-scale system, consider the training scripts at http://data.statmt.org/wmt17_systems/training/

#### `nematus/nmt.py` : use to train a new model

#### data sets; model loading and saving
| parameter | description |
|---        |---          |
| --source_dataset PATH | parallel training corpus (source) |
| --target_dataset PATH | parallel training corpus (target) |
| --dictionaries PATH [PATH ...] | network vocabularies (one per source factor, plus target vocabulary) |
| --saveFreq INT | save frequency (default: 30000) |
| --model PATH, --saveto PATH | model file name (default: model) |
| --reload PATH | load existing model from this path. Set to "latest_checkpoint" to reload the latest checkpoint in the same directory of --saveto |
| --no_reload_training_progress | don't reload training progress (only used if --reload is enabled) |
| --summary_dir PATH | directory for saving summaries (default: same directory as the --saveto file) |
| --summaryFreq INT | Save summaries after INT updates, if 0 do not save summaries (default: 0) |

#### network parameters
| parameter | description |
|---        |---          |
| --embedding_size INT, --dim_word INT | embedding layer size (default: 512) |
| --state_size INT, --dim INT | hidden state size (default: 1000) |
| --source_vocab_sizes INT [INT ...], --n_words_src INT [INT ...] | source vocabulary sizes (one per input factor) (default: None) |
| --target_vocab_size INT, --n_words INT | target vocabulary size (default: -1) |
| --factors INT | number of input factors (default: 1) |
| --dim_per_factor INT [INT ...] | list of word vector dimensionalities (one per factor): '--dim_per_factor 250 200 50' for total dimensionality of 500 (default: None) |
| --enc_depth INT | number of encoder layers (default: 1) |
| --enc_recurrence_transition_depth INT | number of GRU transition operations applied in the encoder. Minimum is 1. (Only applies to gru). (default: 1) |
| --dec_depth INT | number of decoder layers (default: 1) |
| --dec_base_recurrence_transition_depth INT | number of GRU transition operations applied in the first layer of the decoder. Minimum is 2. (Only applies to gru_cond). (default: 2) |
| --dec_high_recurrence_transition_depth INT | number of GRU transition operations applied in the higher layers of the decoder. Minimum is 1. (Only applies to gru). (default: 1) |
| --dec_deep_context | pass context vector (from first layer) to deep decoder layers |
| --use_dropout | use dropout layer (default: False) |
| --dropout_embedding FLOAT | dropout for input embeddings (0: no dropout) (default: 0.2) |
| --dropout_hidden FLOAT | dropout for hidden layer (0: no dropout) (default: 0.2) |
| --dropout_source FLOAT | dropout source words (0: no dropout) (default: 0.0) |
| --dropout_target FLOAT | dropout target words (0: no dropout) (default: 0.0) |
| --use_layer_norm, --layer_normalisation | Set to use layer normalization in encoder and decoder |
| --tie_encoder_decoder_embeddings | tie the input embeddings of the encoder and the decoder (first factor only). Source and target vocabulary size must be the same |
| --tie_decoder_embeddings | tie the input embeddings of the decoder with the softmax output embeddings |
| --output_hidden_activation {tanh,relu,prelu,linear} | activation function in hidden layer of the output network (default: tanh) |
| --softmax_mixture_size INT | number of softmax components to use (default: 1) |

#### training parameters
| parameter | description |
|---        |---          |
| --maxlen INT | maximum sequence length for training and validation (default: 100) |
| --batch_size INT | minibatch size (default: 80) |
| --token_batch_size INT | minibatch size (expressed in number of source or target tokens). Sentence-level minibatch size will be dynamic. If this is enabled, batch_size only affects sorting by length. (default: 0) |
| --max_epochs INT | maximum number of epochs (default: 5000) |
| --finish_after INT | maximum number of updates (minibatches) (default: 10000000) |
| --decay_c FLOAT | L2 regularization penalty (default: 0.0) |
| --map_decay_c FLOAT | MAP-L2 regularization penalty towards original weights (default: 0.0) |
| --prior_model PATH | Prior model for MAP-L2 regularization. Unless using " --reload", this will also be used for initialization. |
| --clip_c FLOAT | gradient clipping threshold (default: 1.0) |
| --label_smoothing FLOAT | label smoothing (default: 0.0) |
| --no_shuffle | disable shuffling of training data (for each epoch) |
| --keep_train_set_in_memory | Keep training dataset lines stores in RAM during training |
| --no_sort_by_length | do not sort sentences in maxibatch by length |
| --maxibatch_size INT | size of maxibatch (number of minibatches that are sorted by length) (default: 20) |
| --optimizer {adam} | optimizer (default: adam) |
| --learning_rate FLOAT, --lrate FLOAT | learning rate (default: 0.0001) |
| --adam_beta1 FLOAT | exponential decay rate for the first moment estimates (default: 0.9) |
| --adam_beta2 FLOAT | exponential decay rate for the second moment estimates (default: 0.999) |
| --adam_epsilon FLOAT | constant for numerical stability (default: 1e-08) |
| --valid_token_batch_size INT | validation minibatch size (expressed in number of source or target tokens). Sentence-level minibatch size will be dynamic. If this is enabled, valid_batch_size only affects sorting by length. (default: 0) |

#### validation parameters
| parameter | description |
|---        |---          |
| --valid_source_dataset PATH | source validation corpus (default: None) |
| --valid_target_dataset PATH | target validation corpus (default: None) |
| --valid_batch_size INT | validation minibatch size (default: 80) |
| --validFreq INT | validation frequency (default: 10000) |
| --valid_script PATH | path to script for external validation (default: None). The script will be passed an argument specifying the path of a file that contains translations of the source validation corpus. It must write a single score to standard output. |
| --patience INT | early stopping patience (default: 10) |

#### display parameters
| parameter | description |
|---        |---          |
| --dispFreq INT | display loss after INT updates (default: 1000) |
| --sampleFreq INT | display some samples after INT updates (default: 10000) |
| --beamFreq INT | display some beam_search samples after INT updates (default: 10000) |
| --beam_size INT | size of the beam (default: 12) |

#### translate parameters
| parameter | description |
|---        |---          |
| --no_normalize | Cost of sentences will not be normalized by length |
| --n_best | Print full beam |
| --translation_maxlen INT | Maximum length of translation output sentence (default: 200) |

#### `nematus/translate.py` : use an existing model to translate a source text

| parameter | description |
|---        |---          |
| -v, --verbose | verbose mode |
| -m PATH [PATH ...], --models PATH [PATH ...] | model to use; provide multiple models (with same vocabulary) for ensemble decoding |
| -b INT, --minibatch_size INT | minibatch size (default: 80) |
| -i PATH, --input PATH | input file (default: standard input) |
| -o PATH, --output PATH | output file (default: standard output) |
| -k INT, --beam_size INT | beam size (default: 5) |
| -n [ALPHA], --normalization_alpha [ALPHA] | normalize scores by sentence length (with argument, exponentiate lengths by ALPHA) |
| --n_best | write n-best list (of size k) |
| --maxibatch_size INT | size of maxibatch (number of minibatches that are sorted by length) (default: 20) |

#### `nematus/score.py` : use an existing model to score a parallel corpus

| parameter | description |
|---        |---          |
| -v, --verbose | verbose mode |
| -m PATH [PATH ...], --models PATH [PATH ...] | model to use; provide multiple models (with same vocabulary) for ensemble decoding |
| -b INT, --minibatch_size INT | minibatch size (default: 80) |
| -n [ALPHA], --normalization_alpha [ALPHA] | normalize scores by sentence length (with argument, exponentiate lengths by ALPHA) |
| -o PATH, --output PATH | output file (default: standard output) |
| -s PATH, --source PATH | source text file |
| -t PATH, --target PATH | target text file |


#### `nematus/rescore.py` : use an existing model to rescore an n-best list.

The n-best list is assumed to have the same format as Moses:

    sentence-ID (starting from 0) ||| translation ||| scores

new scores will be appended to the end. `rescore.py` has the same arguments as `score.py`, with the exception of this additional parameter:

| parameter             | description |
|---                    |--- |
| -i PATH, --input PATH | input n-best list file (default: standard input) |


#### `nematus/theano_tf_convert.py` : convert an existing theano model to a tensorflow model

If you have a Theano model (model.npz) with network architecture features that are currently
supported then you can convert it into a tensorflow model using `nematus/theano_tf_convert.py`.

| parameter | description |
|---        |---          |
| --from_theano | convert from Theano to TensorFlow format |
| --from_tf | convert from Tensorflow to Theano format |
| --in PATH | path to input model |
| --out PATH | path to output model |


PUBLICATIONS
------------

if you use Nematus, please cite the following paper:

Rico Sennrich, Orhan Firat, Kyunghyun Cho, Alexandra Birch, Barry Haddow, Julian Hitschler, Marcin Junczys-Dowmunt, Samuel Läubli, Antonio Valerio Miceli Barone, Jozef Mokry and Maria Nadejde (2017): Nematus: a Toolkit for Neural Machine Translation. In Proceedings of the Software Demonstrations of the 15th Conference of the European Chapter of the Association for Computational Linguistics, Valencia, Spain, pp. 65-68.

```
@InProceedings{sennrich-EtAl:2017:EACLDemo,
  author    = {Sennrich, Rico  and  Firat, Orhan  and  Cho, Kyunghyun  and  Birch, Alexandra  and  Haddow, Barry  and  Hitschler, Julian  and  Junczys-Dowmunt, Marcin  and  L\"{a}ubli, Samuel  and  Miceli Barone, Antonio Valerio  and  Mokry, Jozef  and  Nadejde, Maria},
  title     = {Nematus: a Toolkit for Neural Machine Translation},
  booktitle = {Proceedings of the Software Demonstrations of the 15th Conference of the European Chapter of the Association for Computational Linguistics},
  month     = {April},
  year      = {2017},
  address   = {Valencia, Spain},
  publisher = {Association for Computational Linguistics},
  pages     = {65--68},
  url       = {http://aclweb.org/anthology/E17-3017}
}
```

the code is based on the following model:

Dzmitry Bahdanau, Kyunghyun Cho, Yoshua Bengio (2015): Neural Machine Translation by Jointly Learning to Align and Translate, Proceedings of the International Conference on Learning Representations (ICLR).

please refer to the Nematus paper for a description of implementation differences


ACKNOWLEDGMENTS
---------------
This project has received funding from the European Union’s Horizon 2020 research and innovation programme under grant agreements 645452 (QT21), 644333 (TraMOOC), 644402 (HimL) and 688139 (SUMMA).
