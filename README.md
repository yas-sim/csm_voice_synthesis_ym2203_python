# Composite Sinusoidal Modeling (CSM) Voice Synthesis by Python  

**Disclaimer**: I'm not an expert on FM sound devices.  
**Caveat**: This is just an experimental code.  

## Description:  
Yamaha YM2203C (OPN) equivalent CSM voice synthesis test code.  

![jupyter](./resources/jupyter-csm.png)

|program|description|
|-|-|
|`csm_synthesis_fm.ipynb`|FM-sound device like CSM synthesizer.<br>Reads 'Mutsu' compatible FM-sound register access log file, perform CSM voice synthesis, and playback/save the generated voice data.|
|`csm_to_fm.jpynb`|CSM data to 'Mutsu' access log converter<br>Generates 'Mutsu' FM-sound register access log file compatible data from a `*.csm` data.<br>`*.csm` data can be generated by `csm_encoding.ipynb` in the submodule [`csm_enc_synth`](https://github.com/yas-sim/csm_voice_encode_synthesis_python).|
|`csm_enc_synth/csm_encoding.ipynb`|Audio to CSM data converter.<br>Reads an audio file and generates a CSM data file.|
|`csm_enc_synth/csm_synthesis.ipynb`|CSM synthesizer.<br>Reads a CSM data file and synthesis an audio file from the CSM data|

## Download (git clone)

```sh
git clone https://github.com/yas-sim/csm_voice_synthesis_ym2203_python
cd csm_voice_synthesis_ym2203_python
git submodule init
git submodule update
```

## Workflow (3 types)  
1. Playback CSM voice from old 8-bit games using CSM voice synthesis with a YAMAHA FM-sound device (fm_log -> audio)  
	1. Capture FM-sound device register access log file using 'Mutsu', a Fujitsu FM-7 series emulator (Use `mon fmwrite` command). Save it as `*.log`. Please refer to the [Input data](#input-data-fm-sound-device-register-access-log-file) section for details.    
	2. Playback CSM voice from the FM-sound access log file. (`csm_synthesis_fm.ipynb`)  

2. Normal CSM (audio -> csm -> audio, no FM-sound device restriction)  
	1. Convert an audio file to CSM data file (`csm_enc_synth/csm_encoding.ipynb`)  
	2. Playback CSM voice from the CSM data file (`csm_enc_synth/csm_synthesis.ipynb`)  

3. FM-sound device simulated CSM (audio -> csm -> fm_log -> audio)  
	1. Convert an audio file to CSM data file (`csm_enc_synth/csm_encoding.ipynb`)
	2. Convert CSM data to 'Mutsu' compatible FM-sound register access log file. (`csm_to_fm.ipynb`)
	3. Playback CSM voice from the FM-sound access log file. (`csm_synthesis_fm.ipynb`)

## Input data (FM-sound device register access log file)  

This program assumes the input data in the format below:  

```
YM2203C Reg[$26]=$FB at 21434154442
YM2203C Reg[$27]=$BA at 21434218321
YM2203C Reg[$B2]=$07 at 21434281644
YM2203C Reg[$32]=$01 at 21434348855
YM2203C Reg[$3A]=$01 at 21434412734
```
The last decimal numbers are timestamp [ns].  

You need to prepare the input data like above somehow.  
One of the way is to use the ['Mutsu'](https://github.com/captainys/77AVEMU), a Fujitsu FM-7 series personal computer emulator from captainYS.  
The Mutsu can dump FM sound device register access information with following command.  
```
mon fmwrite
```
You can trim the log data down to pick up only the part for CSM voice synthesis.  
You can identify the required part by checking the register $27 (Ch3 mode). When the Reg27[7:6] are set to '10', the FM sound device is set to CSM voice synthesis mode.  
You might need the prescaler setting change part right before the Ch3 mode change. Otherwise, the frequency of the result sound data might be different from expected.  

**Note**: Most YM2203 datasheet tells Reg27[7:6]='01' for CSM voice synthesis mode, but it should be an error.  
**Note**: I can't share the input data due to copyright.  

```
YM2203C Reg[$A6]=$00 at 21433762838
YM2203C Reg[$A2]=$00 at 21433831160
------------------------------------------- You might need from here
YM2203C Reg[$2F]=$00 at 21433900593  <= Prescaler change to 1/2 (from 1/3)
Pre-Scaler [2F]=00
YM2203C Reg[$25]=$00 at 21433964472
YM2203C Reg[$24]=$00 at 21434027240
YM2203C Reg[$28]=$02 at 21434090563
YM2203C Reg[$26]=$FB at 21434154442
YM2203C Reg[$27]=$BA at 21434218321  <= Reg27[7:6]=='10'
YM2203C Reg[$B2]=$07 at 21434281644
YM2203C Reg[$32]=$01 at 21434348855

  :
  :
  :
YM2203C Reg[$27]=$AB at 23888862699
YM2203C Reg[$28]=$02 at 23888965459
YM2203C Reg[$2D]=$02 at 23889028227
Pre-Scaler [2D]=02
YM2203C Reg[$2E]=$02 at 23889090995
Pre-Scaler [2E]=02
YM2203C Reg[$42]=$7F at 23889154318
YM2203C Reg[$4A]=$7F at 23889217641
YM2203C Reg[$46]=$7F at 23889280964
YM2203C Reg[$4E]=$7F at 23889344287
YM2203C Reg[$92]=$00 at 23889409276
YM2203C Reg[$9A]=$00 at 23889473155
YM2203C Reg[$96]=$00 at 23889537034
YM2203C Reg[$9E]=$00 at 23889598691
------------------------------------------- to here
YM2203C Reg[$27]=$30 at 23889684233  <= Reg27[7:6]=='00'
YM2203C Reg[$28]=$02 at 23889747556
YM2203C Reg[$2D]=$02 at 23889810324
```

### Memo:  
Convert audio data into mono/32Kbps wav format data.

```sh
ffmpeg\bin\ffmpeg.exe -i input.wav -ar 32000 -ac 1 -f wav "output.wav"
```
