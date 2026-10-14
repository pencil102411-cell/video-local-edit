# 平台适配记录

在给出提交设置、排查报错、处理母版帧数、预处理参考图，或者拼接、对比输出时读取。本记录核对于 2026-10-09，平台限额会变，提交前以当前入口为准。

来源：

- [火山方舟《Doubao Seedance 2.5 教程》](https://docs.volcengine.com/docs/ark/seedance-2-5?lang=zh)（页面标注更新于 2026-09-30）
- [火山方舟《Doubao Seedance 2.5 提示词指南》](https://docs.volcengine.com/docs/ark/seedance-2-5-prompt-guide?lang=zh)（页面标注更新于 2026-10-09）
- [《Seedance 2.5 提示词撰写指南》飞书版](https://bytedance.larkoffice.com/docx/OsiUdR1OxoDqvnxsK8LczYx7nPd) 2.4 节“视频编辑”
- [Runware：Editing video — Seedance 2.5](https://runware.ai/docs/models/bytedance-seedance-2-5/guides/editing)（第三方入口）

## 火山方舟 API

模型 ID：`doubao-seedance-2-5-260628`。

**任务判定。** `content` 中至少有一个 `role` 为 `reference_video` 的视频，且提示词表达编辑意图，就会判为视频编辑。参考图和参考音频的 `role` 分别为 `reference_image`、`reference_audio`。

**参数。**

| 参数 | 编辑任务取值 | 说明 |
| --- | --- | --- |
| `omni_reference_task_type` | `edit` | 显式指定后，参数校验前置为同步报错；不传或设为 `auto` 时，由模型判定任务类型 |
| `ratio` | `adaptive` | 必须如此；输出画幅继承待编辑视频 |
| `duration` | `-1` | 必须如此；输出时长基本继承待编辑视频，可能是非整数秒 |
| `output_format` | `mov`（建议） | H.264 + yuv444p + PCM，色彩、亮度和声画一致性更好；Windows 上用 VLC、mpv、ffplay 播放 |
| `resolution` | `480p` / `720p` / `1080p` | 默认 720p；1080p 输出为 10bit H.265 |
| `generate_audio` | 官方编辑示例为 `true` | 只改画面时，输出声音是否与原音完全一致，需要验证 |

**报错。** 指定了 `edit`，但模型按提示词判成了其他类型：`InvalidParameter.TaskTypeMismatch`。参数与任务类型不兼容：`InvalidParameter.TaskTypeConstraint`。提示词里要有编辑触发词（编辑视频、增加/加上、删除/去掉、修改/替换/改成），并避开延长触发词（向前/向后延长、延续、续写）。

**输入视频。**

- 格式 mp4 / mov，视频编码 H.264 / H.265；音频编码 AAC / MP3（mov 还支持 PCM）。
- 编辑任务的单个视频时长 4–30 秒。飞书指南评论区有答复称“多几帧是兼容的”，这不是正文规则，不要依赖。
- 官方建议编辑视频在 20 秒以内，更长时稳定性下降。
- 帧率 24–60；单个文件不超过 200 MB；宽高比 0.4–2.5；边长 300–6000 px；总像素 407,696–8,295,044。
- 最多 10 个视频，**所有视频总时长不超过 30 秒**，所以待编辑视频越长，能附带的参考视频越短。
- 视频只能通过 URL 或素材 ID 传入，需要公网可访问。

**参考图。** 视频参考图编辑时，1–5 张效果较好，6–8 张可以尝试。参考图分辨率不要超过输出分辨率，否则草地、树叶等密集纹理处可能出现指纹状纹理。

**输出时长。** 基本对齐输入，但可能略短：教程写误差不超过 0.4 秒，指南表格写约 0.3 秒，常见问题写约 0.33 秒，差异来自过渡帧压缩。以 Seedance 2.5 生成的视频作为输入时，官方称时长不会有差距；也可以把输入帧数调成 8n+1。确实需要精确时长又不必严格编辑时，可以改用参考生成。

**人脸素材。** Seedance 2.5 不支持直接上传含真人人脸的参考图或视频。本账号下部分模型生成的含人脸**原始产物**可以作为输入，不会触发审核拦截。经过裁剪、拼接、转码的文件是否仍被认作可信产物，本次未核实；需要补手柄或调帧数时，先用小样测试。实测某入口接受了写实人脸三视图，但长相迁移很弱，见下方“实测输出情况”。

**保存期限。** 输出视频 URL 保存 24 小时，最多下载 100 次；任务记录保存 7 天。生成后及时下载。

**样片模式。** `draft: true` 先出 480p 样片，确认后用样片任务 ID 生成 1080p 成片，两步分别计费。官方计费说明提到样片任务“是否包含输入视频”，但编辑任务能否走样片模式，本次没有逐项核实，以当前接口为准。

**声音符号。** 音乐用 `()`，音效用 `<>`，台词用 `{}`，字幕用 `【】`。

## 第三方与英文入口

- **Runware：** 源视频放在 `inputs.video`，提示词中写 `@Video1`；`duration` 必须为 `"auto"`，整数会被拒；画幅继承源视频，不接受 `width`、`height`；可附 `inputs.referenceImages`、`inputs.referenceAudios`；提示词以 remove、replace、relight、restyle、change to 等编辑动词开头；建议输出 MOV，尤其是链式编辑。
- **Scenario：** video-rephotography 记录过其知识库的说法（2026-09-23）：编辑使用视频参考与 Auto 时长，画幅继承源视频，提示词上限 6,000 字符。
- 其他入口要先核实：是否支持编辑任务、引用标签的写法、时长与画幅行为、音轨行为。只支持图生视频的入口，不能当作等效的视频编辑。

## 官方已知问题与对策

| 现象 | 对策 |
| --- | --- |
| 编辑输出比原片短约 0.33 秒 | 输入帧数调成 8n+1；或用 Seedance 2.5 自己的产物作输入；或改用参考生成（内容不再严格一致） |
| 用编辑改画幅，补出来的画面和时序不一致 | 改用参考生成，并逐镜描述时序与画面内容 |
| 一次承担多项参考和编辑要求，效果不稳定 | 拆成多个简单任务，例如先用参考任务生成某个动画，再在编辑任务里引用它 |
| 写了“不要字幕”仍出现字幕 | 母版不要带字幕；台词按“角色台词（情绪）：内容”书写，不在台词后复述词语 |
| 写了“禁止背景音乐”，特效音仍被做成音乐 | 把音乐类同义词列全，在开头和结尾各写一次；或在后期分离人声（会连同音效一起去掉） |
| 情绪描述过强，瞳孔发光 | 最高优先级约束“正常人眼、不发光”，改用平和的情绪词 |
| 参考图引发指纹状纹理 | 参考图分辨率不超过输出分辨率 |
| 音频出现水泡声、回音、啸叫 | 去掉提示词中与水、波浪、回音有关的描述 |
| Seedance 2.0 与 2.5 美学风格差异大 | 官方针对延长任务建议用 mov 保持 2.0 风格；编辑 2.0 产物时同样检查风格漂移（这一条是推断，未经官方验证） |
| 特殊色度采样的 JPG 解码失败 | `ffmpeg -i in.jpg -pix_fmt yuvj420p fixed.jpg`，或转成 PNG |

## 实测输出情况（2026-10-09，用户自己的提交入口，入口名称未记录）

以下是同一入口、同一天、9 条输出上观察到的情况，不是官方规则。详细过程见 [field-notes.md](field-notes.md)。

| 项目 | 观察 |
| --- | --- |
| 容器与分辨率 | 返回 mp4，分辨率与母版相同（992×432）；提交时选了什么分辨率没有记录 |
| 音轨 | AAC 32kHz（母版是 44.1kHz）；对白的时点与原片一致，交付时换回原音 |
| 帧数 | 119 帧母版 → 121 帧，比母版还长；172 帧母版 → 145 帧（四次一致，原因未核实） |
| 切点 | 一次与母版一致（第 53 帧），一次提前到第 52 帧 |
| 写实人脸参考图 | 三视图上传后没有被拦截，但正脸特写的长相没有换成；是否与人脸限制有关未知 |

结论：不要假设输出“只会比母版短 0.3 秒”。拼接或回贴前，先用下面的命令量帧数和切点。

## 本地处理（ffmpeg）

以下命令已在 ffmpeg 9.0.2（Windows 10，PowerShell 5.1）上测试通过。处理前保留原文件，输出另存。

**找切点**（打印切点的时间；阈值 0.2 能抓到硬切，太低会把快速动作也算进去）：

```powershell
ffmpeg -hide_banner -i 视频.mp4 -vf "select='gt(scene,0.2)',metadata=print:key=lavfi.scene_score" -an -f null - 2>&1 | Select-String "pts_time"
```

**同一帧号并排对照**（例：第 30 帧；放大脸可以在 select 后面加 `crop=宽:高:x:y,scale=...`）：

```powershell
ffmpeg -i 母版.mov -i 输出.mp4 -filter_complex "[0:v]select='eq(n\,30)'[a];[1:v]select='eq(n\,30)',scale=992:432[b];[a][b]hstack" -frames:v 1 -fps_mode vfr 对照_30.png
```

ffmpeg 9 已移除 `-vsync`，抽单帧要用 `-fps_mode vfr`。

**换回原音**（画面直接复制）。不要加 `-shortest`：实测与 `-c:v copy` 一起用会把 169 帧截成 166 帧。

```powershell
ffmpeg -i 输出.mov -i 母版.mov -map 0:v -map 1:a -c:v copy -c:a pcm_s16le 成片.mov
```

**按切点拼接两条输出**（例：切点在第 45 帧，A 取切点前，B 取切点后，音轨用母版）：

```powershell
ffmpeg -i A输出.mov -i B输出.mov -i 母版.mov -filter_complex "[0:v]trim=end_frame=45,setpts=PTS-STARTPTS[a];[1:v]trim=start_frame=45,setpts=PTS-STARTPTS[b];[a][b]concat=n=2:v=1:a=0[v]" -map "[v]" -map 2:a -c:v libx264 -crf 12 -pix_fmt yuv420p -c:a pcm_s16le 成片.mov
```

两条输出的切点不在同一帧时，按各自实测的切点分别 trim。

**上下对比视频**（母版在上，输出在下，用母版音轨，带中文标签）：

```powershell
powershell -ExecutionPolicy Bypass -File scripts/make-compare.ps1 -Master 母版.mov -Edited 输出.mp4 -Out 对比.mp4 -TopLabel "原视频" -BottomLabel "测试 01 换背景"
```

脚本取两者中较短的帧数，画布取较大的分辨率。中文标签先写入 UTF-8 文本文件，再用 drawtext 的 `textfile=` 读取，避开 Windows 命令行的编码问题；字体默认是 `C:\Windows\Fonts\msyh.ttc`。

**参考图预处理：**

```powershell
# 1. 检测黑边，输出类似 crop=1304:852:348:2
ffmpeg -hide_banner -loop 1 -t 0.2 -i 参考.webp -vf "cropdetect=limit=30:round=2:reset=0" -f null - 2>&1 | Select-String "crop="
# 2. 去黑边并缩到不超过 1280x720，存成 PNG
ffmpeg -i 参考.webp -vf "crop=1304:852:348:2,scale=1280:720:force_original_aspect_ratio=decrease:flags=lanczos" -frames:v 1 -update 1 图片1_服装.png
# 3. 从三视图里裁出单张侧脸，并水平翻转成与镜头相同的朝向
ffmpeg -i 头部三视图.webp -vf "crop=440:470:1440:140,hflip" -frames:v 1 -update 1 图片2_侧脸.png
```

PNG 文件名可以带中文。单帧输出要加 `-update 1`，否则 ffmpeg 会按序列命名报警。

## 本地帧数处理（ffmpeg）

以下命令已在 ffmpeg 9.0.2 上测试通过。处理前保留原文件，输出另存。

查看帧数、帧率和时长：

```powershell
ffprobe -v error -select_streams v:0 -count_frames -show_entries stream=nb_read_frames,r_frame_rate,duration -of default=nw=1 母版.mp4
```

8n+1 的计算方法：`n = floor((帧数 - 1) / 8)`，目标帧数 = `8n + 1`，最多舍去 7 帧。24 fps 时，“整秒数 × 24 + 1”恰好是 8n+1，例如 5 秒对应 121 帧，10 秒对应 241 帧。

裁到指定帧数（以 161 帧为例），输出 mov：

```powershell
ffmpeg -i 母版.mp4 -frames:v 161 -c:v libx264 -crf 12 -pix_fmt yuv420p -c:a pcm_s16le -shortest 母版_161f.mov
```

实测输出 161 帧；音轨比画面短约 20 毫秒，属于 `-shortest` 的截断误差。裁完用 ffprobe 复核帧数。舍掉的是结尾帧，若编辑窗口靠近结尾，先确认窗口仍在保留范围内。

短于 4 秒的镜头：优先回到剪辑前的完整生成文件取手柄；没有手柄时，在剪辑软件里接上相邻镜头，提示词写明切点和只编辑哪一段。不建议用定格帧凑时长：模型可能把定格的部分也动起来（这一条是推断，未经测试）。

本 skill 自己不提交生成任务。上面的“实测输出情况”来自用户在自己入口的提交，换了入口或账户，要重新核对。
