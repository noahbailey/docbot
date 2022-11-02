# Spellcheck Linux

Hunspell is a simple unix utility to spellcheck text. 

## Install

    sudo apt install hunspell hunspell-en-ca

## Process files

Just run the tool against your text files: 

    hunspell -l notes.txt

This outputs all the misspelled words, one per line. 

