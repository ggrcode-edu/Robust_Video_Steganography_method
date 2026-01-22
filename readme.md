MEC-AQIM (DWT), NDCA
# Paper Links
1. MEC-AQIM (DWT): Adaptive QIM With Minimum Embedding Cost for Robust Video Steganography on Social Networks（https://ieeexplore.ieee.org/document/9924227）
2. NDCA: NDCA: a neighboring block differences-based cost assignment method for robust video steganography on social networks (https://link.springer.com/article/10.1186/s13635-025-00214-6#Sec17)

# Function Overview
MEC_AQIM_tools.exe and NDCA_tools.exe are executable files for two different algorithms. Both tools have the same usage method and command-line interface, only differing in the algorithm. The specific instructions are shown below (using MEC_AQIM_tools.exe as an example).

This tool provides three core functions:

- embed：Embeds a message in a specified cover video, with QIM parameter configuration, RS encoding, STC encoding, and data disorder capabilities.

- extract：Extracts the message from a specified stego video. The parameters must be consistent with those used during embedding.

- compare: Compare the original message npy file with the extracted message npy file to calculate the Bit Error Rate (BER).


# Usage Setup
## Dependencies
* Windows 10/11 (64-bit)
* ffmpeg.exe
* VC++ 2022 Redistributable, located in the dependence folder. Add it to the system environment or copy it to the same directory as the exe files.

## Running
Open Windows Command Prompt (CMD) or PowerShell, navigate to the directory containing the exe files, and execute commands in the following format.


```
## View help (recommended to execute first)
MEC_AQIM_tools -h

## Embed, message file is a random bitstream, a list that can be replaced, loaded using numpy.load, elements are 01, e.g., [0,1,1,1,0,0,1,...].
## After embedding, two files will be generated in the specified temporary directory [variable-dir]: base_orig_encoded_data_bits.npy and base_orig_data_bits.npy.
## base_orig_encoded_data_bits.npy is the actual embedded bits after RS encoding, data shuffling, etc.
## base_orig_data_bits.npy is the original bitstream.
MEC_AQIM_tools embed --cover-vd ./cover/2.mp4 --variable-dir ./tmp_variable --orig-QIM 20,400 --msg-file ./tmp_variable/msg.npy

## Extract, extract the message file to the specified directory [variable-dir]
## Two files will be generated: recovered_encoded_data_bits.npy and recovered_msg_bits.npy
## recovered_encoded_data_bits.npy is the actually extracted bitstream, recovered_msg_bits.npy is the restored bitstream after RS decoding, data recovery, etc.
MEC_AQIM_tools extract --stego-vd ./tmp_variable/2_s.mp4 --adaptive-pcaInterval-QIM-file "./tmp_variable/base_(20, 400)_adaptive_pcaInterval_QIM.npy" --variable-dir ./tmp_variable

## Calculate Bit Error Rate (BER)
## Compare communication success rate: use recovered_msg_bits.npy and base_orig_data_bits.npy
## Calculate Bit Error Rate: use recovered_encoded
MEC_AQIM_tools compare --file1 ./tmp_variable/base_orig_data_bits.npy --file2 ./tmp_variable/base_recovered_msg_bits.npy

```

# Detail
## Embedding
Embed an npy format message into a video carrier to generate a stego video.

```
MEC_AQIM_tools.exe embed 
--cover-vd ./cover/2.mp4                # Cover video, required
--variable-dir ./tmp_variable           # Temporary directory, required
--orig-QIM 20,400                       # Original QIM range parameter, required. MEC_AQIM will search for base_([orig-QIM])_adaptive_pcaInterval_QIM.npy in the [variable-dir] directory. If it exists, it will use the adaptive quantizer from that file. Otherwise, it will generate a new adaptive quantizer based on orig-QIM, which can take a long time.
--msg-file ./tmp_variable/msg.npy       # Message file, required
--QIM-err-threshold-ratio 0.005         # QIM error threshold, default 0.005
--rs-packet-length 60                   # Error correction parameter, packet length, default 60
--rs-msg-length 20                      # Error correction parameter, message length, default 20
--is-stc True                           # Whether to use STC, default True
--is-shuffle True                       # Whether to shuffle data, default True  
--channel-type crf                      # Simulated channel transcoding type, default crf
--channel-param 26                      # Simulated channel transcoding parameter, default 26
--encoderType h264                      # Encoder type, default h264   
--embedding_ratio 0.5                   # Embedding ratio, default 0.5, range 0-0.5
```

## Video Transcoding
ffmpeg -i [stego_video] -y -preset fast -crf 26 [transcoded_video]

Example: ffmpeg -i 2_s.mp4 -y -preset fast -crf 26 2_s_26.mp4

## Extract
1. Extract the embedded message from a stego video and generate an npy format file.
2. Two files will be generated: recovered_encoded_data_bits.npy and recovered_msg_bits.npy.
3. recovered_encoded_data_bits.npy is the actually extracted bitstream.
4. recovered_msg_bits.npy is the restored bitstream after RS decoding, data recovery, etc.

```
MEC_AQIM_tools.exe extract 
--stego-vd ./tmp_variable/2_s.mp4       # Stego video, required
--adaptive-pcaInterval-QIM-file ./tmp_variable/base_(20, 400)_adaptive_pcaInterval_QIM.npy # Adaptive quantizer, required. This file is generated during the embedding process based on the set orig-QIM. The embedding and extraction processes must use the same quantizer to maintain accuracy.
--variable-dir ./tmp_variable           # Temporary directory, required. Must be consistent with the directory used during embedding.
--rs-packet-length 60                   # Error correction parameter, packet length, default 60. Must be consistent with the embedding process.
--rs-msg-length 20                      # Error correction parameter, message length, default 20. Must be consistent with the embedding process.
--is-stc True                           # Whether to use STC, default True
--is-shuffle True                       # Whether to shuffle data, default True     
--channel-type crf                      # Simulated channel transcoding type, default crf
--channel-param 26                      # Simulated channel transcoding parameter, default 26
--encoderType h264                      # Encoder type, default h264       
```

## Calculate Bit Error Rate (BER)
Compare the bit error rate of the extracted message.

```
MEC_AQIM_tools.exe compare
--file1 ./tmp_variable/base_orig_data_bits.npy  # Message file 1  
--file2 ./tmp_variable/base_recovered_msg_bits.npy   # Message file 2
```

## Robustness Explanation
1. The MEC-AQIM algorithm uses an adaptive quantizer, which requires adaptation to the video content and video transcoding parameters. Therefore, when changing videos or transcoding parameters, the adaptive quantizer must be regenerated. Especially when transcoding parameters change, such as from CRF18 to CRF26, the adaptive quantizer also needs to be replaced. This can be achieved by modifying the channel-type and channel-param during embedding. NDCA uses a quantization index modulation method, requiring only one QIM step length to be set. However, to achieve similar robustness, the same QIM step length adaptive search algorithm is used, and a weighted sum is obtained to simulate similar robustness.
2. The orig-QIM parameter is a search range, and the adaptive quantizer will search for the optimal quantization parameters within this range. Generally, setting it to 20,400 is sufficient.
3. To test robustness against attacks, you can compress the stego video using ffmpeg.
4. The adaptive quantizer generated by the MEC_AQIM method is saved as base_(20, 400)_adaptive_pcaInterval_QIM.npy. The (20, 400) is determined by orig_qim.
5. The adaptive quantizer generated by the NDCA method is saved as advancedStc_(20, 400)_adaptive_pcaInterval_QIM.npy. The (20, 400) is determined by orig_qim.

## Other Notes
1. The generated stego video uses CRF0 compression, which may not be playable with the default Windows media player. It is recommended to use PotPlayer for playback.
2. This method uses a multi-process approach to accelerate computation, approximately occupying 90% of CPU resources. Higher CPU core count can improve computation speed.
