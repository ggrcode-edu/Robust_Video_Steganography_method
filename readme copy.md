MEC-AQIM (DWT), NDCA
# 论文链接
1. MEC-AQIM (DWT): Adaptive QIM With Minimum Embedding Cost for Robust Video Steganography on Social Networks（https://ieeexplore.ieee.org/document/9924227）
2. NDCA: NDCA: a neighboring block differences-based cost assignment method for robust video steganography on social networks (https://link.springer.com/article/10.1186/s13635-025-00214-6#Sec17)

# 功能概述
MEC_AQIM_tools.exe和NDCA_tools.exe是两种算法的可执行文件。两种工具的使用方法及命令行是一样的，只有算法不同，具体指令如下所示（以MEC_AQIM_tools.exe为例）.

该工具提供三大核心功能：

- embed：将npy格式的消息嵌入指定视频载体，支持QIM参数配置、RS编码、STC编码、数据置乱等特性

- extract：从隐写视频中提取嵌入的消息，需与嵌入时的参数保持一致

- compare：对比原始消息与提取消息的npy文件，计算比特错误率（BER）

# 使用设置
## 环境依赖
* 操作系统：Windows 10/11（64位）
* ffmpeg.exe
* VS2022 的 VC++ 2022运行时库，见dependence文件夹，将其添加到系统环境中，或者复制到与exe文件相同的目录下（本文采用了失真最小化框架STC，采用了MSVC进行编译）

## 运行方式
打开Windows命令提示符（CMD）或PowerShell，进入exe文件所在目录，执行以下格式的命令。


```
\## 查看帮助（推荐先执行）
MEC_AQIM_tools -h

\## 嵌入，消息文件为随机比特流，一个列表，可以替换，使用numpy.load进行加载，元素为01，例[0,1,1,1,0,0,1,...]。
\## 嵌入后会生成两个文件，在指定临时目录[variable-dir]下会生成两个文件base_orig_encoded_data_bits.npy和base_orig_data_bits.npy
\## 其中base_orig_encoded_data_bits.npy是经过RS编码、数据置乱等处理后的实际嵌入比特，base_orig_data_bits.npy是原始的比特流。
MEC_AQIM_tools embed --cover-vd ./cover/2.mp4 --variable-dir ./tmp_variable --orig-QIM 20,400 --msg-file ./tmp_variable/msg.npy

\## 提取，提取消息文件至指定文件夹中[variable-dir]
\## 会生成两个文件，recovered_encoded_data_bits.npy和recovered_msg_bits.npy
\## 其中recovered_encoded_data_bits.npy是实际提取的比特流，recovered_msg_bits.npy是经过RS解码、数据恢复等处理后恢复出的比特流。
MEC_AQIM_tools extract --stego-vd ./tmp_variable/2_s.mp4 --adaptive-pcaInterval-QIM-file "./tmp_variable/base_(20, 400)_adaptive_pcaInterval_QIM.npy" --variable-dir ./tmp_variable

\## 计算误码率
\## 比较通信成功率：使用recovered_msg_bits.npy和base_orig_data_bits.npy
\## 比特误码率：使用recovered_encoded_data_bits.npy和base_orig_encoded_data_bits.npy
MEC_AQIM_tools compare --file1 ./tmp_variable/base_orig_data_bits.npy --file2 ./tmp_variable/recovered_msg_bits.npy

```

# 详细说明
## 嵌入
将npy格式的消息嵌入视频载体，生成隐写视频。

```
AQIM_tools.exe embed 
--cover-vd ./cover/2.mp4                # 载体视频，必选
--variable-dir ./tmp_variable           # 临时目录，必选
--orig-QIM 20,400                       # 原始QIM范围参数，MEC_AQIM方法会在这个范围中构建自适应量化器，必选。MEC_AQIM会搜索[variable-dir]目录下是否存在base_([orig-QIM])_adaptive_pcaInterval_QIM.npy文件，若存在则使用该文件的自适应量化器。否则会根据orig-QIM生成自适应量化器，花费的时间比较久
--msg-file ./tmp_variable/msg.npy       # 消息文件，必选
--QIM-err-threshold-ratio 0.005         # QIM误差阈值，默认0.005
--rs-packet-length 60                   # 纠错码参数，片段长度，默认60
--rs-msg-length 20                      # 纠错码参数，消息长度，默认20
--is-stc True                           # 是否使用失真最小化框架STC，默认True
--is-shuffle True                       # 是否对数据进行置乱，默认True  
--channel-type crf                      # 模拟信道编码类型，默认crf
--channel-param 26                      # 模拟信道编码参数，默认26
--encoderType h264                      # 编码器类型，默认h264  
--embedding_ratio 0.5                   # 负载率，默认0.5，范围在0-0.5
```

## 视频转码
ffmpeg -i [stego_video] -y -preset fast -crf 26 [transcoded_video]

例：ffmpeg -i 2_s.mp4 -y -preset fast -crf 26 2_s_26.mp4

## 提取
1. 从隐写视频中提取嵌入的消息，生成npy格式文件。
2. 会生成两个文件，recovered_encoded_data_bits.npy和recovered_msg_bits.npy
3. 其中recovered_encoded_data_bits.npy是实际提取的比特流，recovered_msg_bits.npy是经过RS解码、数据恢复等处理后恢复出的比特流。

```
AQIM_tools.exe extract 
--stego-vd ./tmp_variable/2_s.mp4       # 载密视频，必选
--adaptive-pcaInterval-QIM-file ./tmp_variable/base_(20, 400)_adaptive_pcaInterval_QIM.npy # 自适应量化器，必选，在嵌入过程中会根据设置的orig-QIM生成该文件，嵌入和提取必须一致，否则提取准确率会受影响
--variable-dir ./tmp_variable           # 临时目录，必选，需要和嵌入时保持一致
--rs-packet-length 60                   # 纠错码参数，片段长度，默认60，必须与嵌入保持一致
--rs-msg-length 20                      # 纠错码参数，消息长度，默认20
--is-stc True                           # 是否使用失真最小化框架STC，默认True
--is-shuffle True                       # 是否对数据进行置乱，默认True      
--channel-type crf                      # 模拟信道编码类型，默认crf
--channel-param 26                      # 模拟信道编码参数，默认26
--encoderType h264                      # 编码器类型，默认h264      
```

## 计算误码率
对比提取消息的误码率

```
AQIM_tools.exe compare
--file1 ./tmp_variable/base_orig_data_bits.npy  # 消息文件1 
--file2 ./tmp_variable/recovered_msg_bits.npy   # 消息文件2
```

## 鲁棒性说明
1. MEC-AQIM算法使用了自适应量化器，自适应量化器的生成需要适配视频内容和视频编码参数。因此，当更换视频以及视频编码参数时，需要重新生成自适应量化器。尤其是视频编码参数变化时，比如从CRF18变成CRF26，自适应量化器也需要更换。可以通过修改嵌入时的channel-type和channel-param来实现。NDCA使用的是量化索引调制方法，只需要设置一个QIM步长即可，但是为了获得类似的鲁棒性，采用了同样的QIM步长自适应搜索算法，然后使用加权求和获得QIM步长以模拟类似的鲁棒性。
2. orig-QIM参数是一个搜索范围，自适应量化器会在该范围内搜索最佳的量化参数。一般设置为20,400即可
3. 测试鲁棒性攻击时，可以将隐写视频使用ffmpeg进行压缩
4. MEC_AQIM方法生成的自适应量化器默认保存为base_(20, 400)_adaptive_pcaInterval_QIM.npy。其中(20, 400)由orig_qim决定
5. NDCA方法生成的自适应量化器默认保存为advancedStc_(20, 400)_adaptive_pcaInterval_QIM.npy。其中(20, 400)由orig_qim决定

## 其他说明
1. 生成的隐写视频使用的是crf0压缩，windows自带播放器可能无法播放，推荐使用potplayer进行播放
2. 该方法采用多进程方法加速计算，大约会占用90%的cpu资源，较高的cpu核数可以提高计算速度
