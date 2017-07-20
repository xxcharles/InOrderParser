# InOrderParser

This implementation is based on the [cnn library](https://github.com/clab/cnn-v1) for this software to function. The reference paper is "In-Order Transition-based Constituent Parsing System"

#### Building

    mkdir build
    cd build
    cmake .. -DEIGEN3_INCLUDE_DIR=/path/to/eigen
    make

#### Data
We borrow the code [get_oracle.py](https://github.com/clab/rnng/blob/master/get_oracle.py) to get top-down oracle
 
    ./get_oracle.py [training data in bracketed format] [training data in bracketed format] > [training top-down oracle]
    ./get_oracle.py [training data in bracketed format] [development data in bracketed format] > [development top-down oracle]   
    ./get_oracle.py [training data in bracketed format] [test data in bracketed format] > [test top-down oracle]

, and then compile pre2mid.cc to get pre2mid to convert them into in-order oracle

    g++ pre2mid.cc -o pre2mid
    ./pre2mid [training top-down oracle] > [training oracle]
    ./pre2mid [development top-down oracle] > [development oracle]
    ./pre2mid [test top-down oracle] > [test oracle]

If you require the related data, contact us.

#### Training

Ensure the related file are linked into the current directory.

    mkdir model/
    ./build/impl/Kparser --cnn-mem 1700 --training_data [training oracle] --dev_data [development oracle] --bracketing_dev_data [development data in bracketed format] -P -t --pretrained_dim 100 -w [pretrained word embeddings] --lstm_input_dim 128 --hidden_dim 128 -D 0.2

#### Test
    
    ./build/impl/Kparser --cnn-mem 1700 --training_data [training oracle] --test_data [test oracle] --bracketing_dev_data [test data in bracketed format] -P --pretrained_dim 100 -w [pretrained word embeddings] --lstm_input_dim 128 --hidden_dim 128 -m [model file]

The automatically generated file test.eval is the result file.

We provide the trained models: [English model](https://drive.google.com/file/d/0B1VhP65vISjoWmNjN0pfTmh5Vnc/view?usp=sharing) and pretrained word embeddings [sskip.100.vectors](https://drive.google.com/open?id=0B1VhP65vISjoZ3ppTnR3YXRMd1E); [Chinese_model](https://drive.google.com/open?id=0B1VhP65vISjocW5fWmdEUVNxY3M) and pretrained word embeddings [zzgiga.sskip.80.vectors](https://drive.google.com/open?id=0B1VhP65vISjoeGJsX2syOGhLWnc)

#### Sampling

    ./build/impl/Kparser --cnn-mem 1700 --training_data [training oracle] --test_data [test oracle] --bracketing_dev_data [test data in bracketed format] -P --pretrained_dim 100 -w [pretrained word embeddings] --lstm_input_dim 128 --hidden_dim 128 -m [model file] --alpha 0.8 -s 100 > samples.act
    ./mid2tree.py samples.act > samples.trees

The samples.props could be fed into following reranking components. 

#### Easy Usage

Download the [model](https://drive.google.com/open?id=0B1VhP65vISjoa014Ul9pS3ZZMU0) for English.

    ./build/impl/Kparser-standard --cnn-mem 1700 --model_dir model -w sskip.100.vectors --train_dict train_dict < [stdin] > [stdout]

The standard input should follow the fomart, Word1 POS1 Word2 POS2 ... Wordn POSn. The example is

    No RB , , it PRP was VBD n't RB Black NNP Monday NNP . .

The standard output is tree in bracketed format.

   (S (INTJ (RB No)) (, ,) (NP (PRP it)) (VP (VBD was) (RB n't) (NP (NNP Black) (NNP Monday))) (. .)) 

If you want to sample trees, you should added --samples [number of samples] --a [alpha], for example, --sample 100 --a 0.8


#### Contact

Jiangming Liu, jmliunlp@gmail.com
