This project implements a complete forced-alignment pipeline using the Montreal Forced Aligner (MFA) on a Windows system through Windows Subsystem for Linux (WSL). A dedicated Conda environment was used to install and run MFA. Pretrained English acoustic and pronunciation dictionary models (english_us_arpa) were selected for alignment. The speech corpus was organized according to MFA requirements, with each audio file paired with a corresponding transcript file. The transcripts were normalized by removing formatting inconsistencies, converting text to uppercase, and ensuring compatibility with the pronunciation dictionary. Vocabulary coverage was verified by comparing transcript words against the dictionary, and six out-of-vocabulary (OOV) words—DEPOLITICIZE, DUKAKIS, MAFFY, MELNICOVE, SJCS, and WBURS—were identified. These OOV words were resolved by generating pronunciations using MFA’s grapheme-to-phoneme (G2P) model and appending them to the base dictionary. After confirming complete dictionary coverage, forced alignment was performed, successfully producing word- and phoneme-level alignments for all utterances in the corpus.
# Activate environment
conda activate mfa

# Install MFA
conda install -c conda-forge montreal-forced-aligner

# Download pretrained models
mfa model download acoustic english_us_arpa
mfa model download dictionary english_us_arpa

# Inspect dictionary
mfa model inspect dictionary english_us_arpa

# Clean transcripts
sed -i ':a;N;$!ba;s/\n/ /g' *.txt
tr '[:lower:]' '[:upper:]' < file.txt > cleaned.txt

# Extract vocabulary
awk '{ for (i=1;i<=NF;i++) print $i }' corpus/*.txt > all_words.txt
sort -u all_words.txt > all_words_sorted.txt

# Extract dictionary words
awk '{print $1}' english_us_arpa.dict > dict_words_raw.txt
sort -u dict_words_raw.txt > dict_words_sorted.txt

# Identify OOV words
comm -23 all_words_sorted.txt dict_words_sorted.txt > oov_words.txt

# Generate pronunciations for OOVs
mfa g2p oov_words.txt english_us_arpa oov_prons.dict
cat oov_prons.dict >> english_us_arpa.dict

# Verify audio
soxi corpus/F2BJRLP1.wav

# Run forced alignment
mfa align corpus english_us_arpa.dict english_us_arpa aligned_output --clean --overwrite



