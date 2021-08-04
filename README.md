# siglearn
Code for BH21 talk: "Generating YARA Rules by Classifying Malicious Byte Sequences"

## Requirements
Requires numpy, pytorch, and boto3.  boto3 is only used to grab training binaries from s3.  That's where mine were stored, but you can load samples from the filesystem just as well.

Trained on pytorch==1.8.1 and numpy 1.20.3.

## Signatures
Pre-computed signatures are available in the 'sigs/' directory.  Performance summaries below:

PE: 1000 signatures.  0.06% FPR on VT benign corpus; 85.63% TPR on VT malicious corpus.  Samples collected from 2020 to 2021.

ELF: 968 signatures.  Tuned for 0% FPR on an Ubuntu corpus; 0.14% FPR on VT benign corpus; 85.10% TPR on VT malicious corpus.

MachO: 715 signatures.  0.4% FPR on VT benign corpus; 97.77% TPR on VT malicious corpus.

These sigs perform a little better than the numbers quoted in the BH talk.  What a difference a month makes!

## Usage
### Generating signatures
Using generate_sigs.py, you can generate signatures using a pre-trained model on a corpus of your own malware.  Example usage:
```
~/siglearn$ python3 generate_sigs.py --model_path models/pe_model.mdl --score_threshold 4 --sample_path samples --yara_filename pe_test.yar
loading model models/pe_model.mdl...
done
attempting to extract sigs for 11 samples...
samples/068c4a7fb3799a52336307e72c3d00d698f7db8ff07746c260e3c691a761141e best sig (score=22.514, offset=00002024): {b'_^[\x89\xec]\xc3\x90\x90\x90\x90\x90UT]\x83\xecDSVW\x8d}\xbc\xbe\xc8'}
samples/002dad23dfb88207e1d9da4960fe1d7eec06637e01a1a16d958bd2951e62b391 best sig (score=10.581, offset=00003d70): {b'[\xc9\xc3`\x90\x90\x90\x90\x90\xb8\x00\x10@\x00\x90\xbb\\\x91@\x00\x90\xb9r`\x8fC'}
samples/0bafea2aca2380df86a4ff5dd6ac588543cb635a8addb60fc21d19ec1df85e58 best sig (score=13.327, offset=0001a718): {b'\x9c@@\x00\xe4=@\x00\x00>@\x00<>@\x00\xec\xb3A\x00\x0eTProp'}
samples/0a792b71efbe61aa3eb866a2c5da4671d8f35be8ef203ef2adc363af082a5245 best sig (score=9.852, offset=0000c561): {b'@\x00\x90p\xc5@\x00\x9c\xc5@\x00\xc0\xc5@\x00#\xd1\x8a\x06\x88\x07\x8aF\x01\x88G'}
samples/0149c6019f5bb592b6f1fa1fb321a12ffc6bcef65f29a97c2f475562ea1a2046 best sig (score=16.678, offset=00025737): {b'\x00\xc23@\x00\x025@\x00\xdc4@\x0045@\x00\x929@\x00\x929@\x00\xe2'}
samples/0f26e5b8d7332005eda228a33fe59636b64f806be169dd3014d9229813979abb best sig (score=21.868, offset=00075f05): {b'fjnbvnnvbvfvvnrnrfjbbvbnbr'}
samples/0d60e158f543cc6713c0e12d8c7fc5e67c81a8075a8d551a2a74d111511f32b7 best sig (score=16.256, offset=000b213e): {b'HjQkQjrWBpbJwHxisrQubMpKNL'}
samples/033cc1fba7e8cc1f1c569ee777e9199b323a6a78cd8b787cc0070e932433e921 best sig (score=16.590, offset=0000007d): {b'\x00\x00\x00e\r\xa3\x87!l\xcd\xd4!l\xcd\xd4!l\xcd\xd4\xafs\xde\xd4+l\xcd'}
samples/0bc7f66fee7e3903aca78b22a14b9dafee7110612a58f88f6833eda2ecf3b200 best sig (score=19.023, offset=0001bbe9): {b"\xc5B\xaf\x85\xb9\x94\xf3\x8e\t\xd3\xcb\x9f\x88'G\xbb\x1dtT\xb8\xd1\xe3\xb8/\xfe\x7f"}
samples/07491fe6421744ea1402f23d01bd2bc5e582e3375e08e9ce713b2239766c0466 best sig (score=0.829, offset=000058be): {b'\xff\x89\x9d\x90\xf4\xff\xfft\x0c\xff\xb5d\xfd\xff\xff\xff\x15,\x10\x00\x01\xc7\x85\xa8\xf4\xff'}
samples/0d891622b8f23824e1cfbd915a61f1ad130c601bf09fea0edb4223f2daaf4705 best sig (score=4.303, offset=0000054c): {b'\x10\x00\x00\xa1PQ@\x00\x85\xc0tX\xa3P0@\x00\x8b\x15\xdca@\x00\x85\xd2\x0f'}
signature success rate: (10/11)
writing out yara rule to pe_test.yar
```

Give the script the path to the pre-trained model, a score threshold, a path to binaries you'd like to extract signatures for, and a filename for the resulting yara signature.  Experimentally, I've found that a threshold above 4 minimizes most false positives, though your mileage may vary.

There's also a 'verbose' option, where it'll dump out all strings that exceed the specified threshold.  This sometimes outputs a lot of signatures, so it's best to run this on one sample at a time.


### Training models
The script for training models lives in 'train.py'.  This is useful if you have >10k malicious samples and >100k benign files to train on.

--uripath:
The 'uripath' file is a simple CSV consisting of a URI (filepath or s3 key) and a label.  Eg.,
```
/home/ubuntu/samples/elf/000008eba10456c6389871e196cc7ba30b9c69856aa5db9e8d4dc10de38f5e89,0
/home/ubuntu/samples/elf/000009d90c7c817ba569930773aa59fea1b81a1ea57bb307afdd544f9f190d55,0
/home/ubuntu/samples/elf/00000e5133e2831c3bc538dd39b9b5a7b889ee8b9ab3df77a9d0b5ac04ed3554,0
/home/ubuntu/samples/elf/00000e689a72065ac7646d1814e0512a33652a30776f99f277cad96df27cf0bc,0
```

--artifactpath:
A directory that holds tensorboard and model artifacts.  Point tensorboard at this directory for accuracy info as the model trains.  Models are saved here every 1000 iterations.

--lr:
Model learning rate

--kernelsz:
Kernel size for convolutional layers.  Larger kernel sizes result in longer signatures.

--embed_dim:
Embedding dimension for projecting input bytes into vector space.

--seqlen:
Size to chunk up samples.  If you're running into OOM's on your GPU, try lowering this.

--topk_counts:
Number of max output units to backprop through.

--architecture:
Number of hidden units for convolutional layers.  More convolutional layers results in longer signatures and larger memory usage.
