# 3DGS 调研 · 幕景 Scendance

面向活动策划三维布置：splat 适合「场地背景观感」，不适合可替换物件主路径。采集→重建→压缩→Web 播的有据对照与 MVP。

> 来源：Grok Bot 调研 Page（canvas `b9d19ff9`，2026-10-25）。本文件按「调研/规格上 GitHub」沉淀，便于队友检索。

## 一句话结论

多视角视频/照片经 COLMAP（或消费级 App）训成 3DGS → 以 `.ply` 为源、转 SOG/SPZ/`.ksplat` 压缩 → 用 Spark 或 SuperSplat 在浏览器播；进幕景只作静态场地背景层，「布置」物件仍走 glTF 网格。

判断依据：经典 3DGS 输出不可编辑实体网格；Three.js 侧 Spark/GaussianSplats3D 可与 mesh 同场景渲染，但 splat 内无法像节点一样替换单件家具；SuGaR/2DGS 提 mesh 可辅助，仍非布置主资产管线。

## ① 开源模型/管线

数字均来自各仓库 README/项目页；无出处写「待核」。

| 方案 | 输入 | 输出 | 许可 | 显存/时长 | 链接 |
| --- | --- | --- | --- | --- | --- |
| 经典 3DGS (INRIA) | 多图/视频 + COLMAP SfM | `.ply`（点云式高斯） | 研究/评估非商用；商用需联系 Inria | 论文质量约 24GB VRAM；训练时长量级待核 | https://github.com/graphdeco-inria/gaussian-splatting |
| 2DGS | 同 3DGS（COLMAP） | splat + 可导出 mesh（TSDF） | 跟随上游 3DGS 许可习惯；代码公开 | 训练加速约 30–40%（作者声明）；绝对显存待核 | https://github.com/hbb1/2d-gaussian-splatting |
| SuGaR | COLMAP 场景；可接已训 3DGS | hybrid mesh+Gaussians；`.ply` / `.obj` | 基于 3DGS 包装；商用需注意上游许可 | 提 mesh 约 30 分钟/单卡（作者 README） | https://github.com/Anttwo/SuGaR |
| Scaffold-GS | COLMAP / 公开数据集 | anchor + MLP 属性；viewer 专用结构 | 跟随 3D-GS LICENSE | 显存/时长待核 | https://github.com/city-super/Scaffold-GS |
| 4DGS (hustvl) | 多视角/时序视频（D-NeRF 等） | 动态 4D 高斯；可导每帧 3DGS | Apache-2.0（仓库标注） | D-NeRF ~8 min；HyperNeRF ~30 min（作者更正） | https://github.com/hustvl/4DGaussians |
| 压缩：SOG / SPZ / `.ksplat` | 已训 `.ply` | SOG（约 15–20×）；SPZ（约 10×）；`.ksplat` | 格式开放；工具各异 | 体积数字来自 PlayCanvas 文档 | https://developer.playcanvas.com/user-manual/gaussian-splatting/formats/ |
| DreamGaussian | 单图 / 文本（SDS） | `.ply` → mesh（`.obj`/`.glb`） | 仓库含 3DGS 衍生许可文件 | 高斯阶段 ~500 iter ~1min；mesh ~1min（README） | https://github.com/dreamgaussian/dreamgaussian |
| LGM | 单图/文本（多视图高斯） | `.ply`；可转 mesh | MIT | 推理约 10GB 显存（README） | https://github.com/3DTopia/LGM |
| Splatter Image | 单图（开类别 Objaverse 等） | 3DGS / `.ply` | BSD-3-Clause | 训练/推理绝对数字待核；有 HF Demo | https://github.com/szymanowiczs/splatter-image |

论文页：https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/ · HF 权重示例 LGM https://huggingface.co/ashawkey/LGM · Splatter Image Demo https://huggingface.co/spaces/szymanowiczs/splatter_image

## ② 产品/平台

价格以公开定价页为准；动态商店页未解析出的标「待核」。

| 产品 | 采集 | 实时? | 导出 | 浏览器播 | 价格档 | 链接 |
| --- | --- | --- | --- | --- | --- | --- |
| Luma 3D Capture | iPhone 视频/环绕（无需 LiDAR） | 云端重建，非本地实时训 | NeRF/Splat → Unreal/Unity 等；具体扩展名待核 | 是（Web 分享 Interactive Scene） | App Store 标 Free；lumalabs.ai/pricing 现为 Dream Machine 积分档，Capture 单独价待核 | https://lumalabs.ai/ios |
| Polycam | iOS/Android/Web；照片/视频；LiDAR Space | 云端 | Splat PLY；mesh glTF/OBJ 等 | 是（托管 viewer） | Free $0（无 splat）；Basic $30/月；Business $400/年·人 | https://poly.cam/pricing |
| Postshot (Jawset) | Win 本地照片/视频；可导 COLMAP | 本地 GPU 训 | PLY / SPZ（付费）；HTML web 导出 | HTML 导出可浏览器开 | Free（非商用、不可导 radiance）；Indie/Studio 付费（商店动态标价，具体欧元待核） | https://www.jawset.com/shop/pricing/ |
| Nerfstudio | 自备图/视频 + COLMAP 等 | 研究管线；非消费级 | 含 Splatfacto → PLY | 自带 Web viewer（训练侧） | 开源 Apache-2.0；自备 GPU | https://github.com/nerfstudio-project/nerfstudio |
| Scaniverse | 手机端捕获 | 设备端训 splat（PlayCanvas 文档） | PLY | 有 Web/App | Freemium（细节待核） | https://developer.playcanvas.com/user-manual/gaussian-splatting/creating/recommended-tools/ |

PlayCanvas 工具对照：https://developer.playcanvas.com/user-manual/gaussian-splatting/creating/recommended-tools/ · INRIA 参考实现明确非商用。

## ③ Web 播放

幕景主栈为 Three.js，优先 Spark；SuperSplat 作剪辑/发布旁路。

| 播放器 | 引擎 | 格式 | 嵌 Three 场景 | 许可/备注 | 链接 |
| --- | --- | --- | --- | --- | --- |
| Spark | THREE.js | `.ply` / `.splat` / `.ksplat` / `.spz` / `.sog` | 是（与 mesh 同场景；动态效果） | 开源；World Labs 维护；antimatter15 推荐继任 | https://sparkjs.dev/ |
| mkkellogg GaussianSplats3D | THREE.js | `.ply` / `.splat` / `.ksplat` | 是（threeScene / DropInViewer） | 作者声明不再积极开发，推荐改用 Spark | https://github.com/mkkellogg/GaussianSplats3D |
| antimatter15/splat | WebGL1 裸实现 | `.ply` → `.splat` | 弱（独立页为主） | 轻量；丢高阶 SH | https://github.com/antimatter15/splat |
| SuperSplat | PlayCanvas / WebGPU | `.ply` / `.splat` / `.spz` / `.sog` 等 | PlayCanvas 嵌入；非原生 Three | MIT 系开源编辑+发布 | https://github.com/playcanvas/supersplat |
| Babylon + KHR_gs | Babylon.js | glTF KHR_gaussian_splatting；Sandbox 还认 `.ply`/`.sog`/`.spz`/`.splat` | 否（Babylon 生态） | Khronos 扩展路径；与 glTF 网格同文件可行 | https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_gaussian_splatting |

压缩交付建议：源 `.ply` → SplatTransform/编辑器转 SOG（约 15–20×，PlayCanvas 文档）再 CDN；大场景考虑 Streamed SOG。

## 推荐 MVP 路径

1. **采集**：手机环绕拍场地（固定曝光、慢移）；或 Polycam Basic 拍 splat。Luma Capture 可作快速预览路径（导出细节待核）。
2. **重建**：消费级：Polycam/Postshot Indie 出 PLY。自托管：COLMAP + 商用友好训练器（gsplat/nerfstudio Splatfacto 等）；勿直接商用 INRIA 参考实现。
3. **压缩**：PLY → SOG 或 SPZ/`.ksplat`；用 SuperSplat 裁 floater、限球谐阶。
4. **Web 播**【候选】：幕景内嵌 Spark SplatMesh，与现有 Three 相机/灯光同场景；或先 iframe SuperSplat 验收观感。
5. **能否进布置**【候选】：仅作锁定背景层（半透明碰撞用粗糙代理 mesh/点云）；家具/展台/灯光仍 glTF 实例化替换。勿把整场 splat 当可编辑场景图。

> **候选进幕景**：Spark 背景层 + 代理碰撞；物件替换不碰 splat。若需「扫完就能量尺寸改墙」，优先 LiDAR/摄影测量 mesh（Polycam Business 平面图），不是纯 3DGS。

## 不适合清单（相对幕景）

- 把整场 splat 当作可布置场景图：无法按物件节点替换桌椅/展台。
- 依赖 splat 做精确动线/净空/碰撞：无稳定表面拓扑；量测应走 mesh/平面图。
- 单图/少图生成（DreamGaussian/LGM/Splatter Image）当场地真相：适合道具概念，不适合真实展厅尺度。
- 4D/动态 GS 做活动布置主路径：采集与播放成本高，超出静态展陈需求。
- 未获授权商用 INRIA gaussian-splatting 参考实现。
- 期望 splat 内局部材质/灯光可编辑等价于 glTF PBR 工作流。

相对 glTF：网格可实例化、替换、UV/材质；splat 是外观场，编辑粒度是裁剪/透明度，不是物件级。

## 与队友边界

1. 几何/CAD、平面图→可布置空间：归「平面图转3D」队友。
2. 灯光氛围、实时观感调色：归「场景氛围」队友。
3. 本页主责：splat 采集→训→压缩→Web 播管线本身，以及「背景层 vs 布置层」边界。

## 关键源

- [3DGS 官方实现](https://github.com/graphdeco-inria/gaussian-splatting)
- [Splat 格式（PLY/SOG/SPZ）](https://developer.playcanvas.com/user-manual/gaussian-splatting/formats/)
- [Polycam 定价](https://poly.cam/pricing)
- [Postshot 定价](https://www.jawset.com/shop/pricing/)
- [Spark THREE.js](https://sparkjs.dev/)
- [SuperSplat](https://github.com/playcanvas/supersplat)
- [Luma Capture App](https://lumalabs.ai/ios)
- [SuGaR](https://github.com/Anttwo/SuGaR)

仍不确定：Postshot Indie/Studio 欧元现价（商店页动态）；Luma Capture 与 Dream Machine 积分定价的关系；各消费级 App 导出文件扩展名完整列表；幕景内 Spark 与导航/选择器深度冲突的实测数据。

## 硬钉（产品侧）

- splat = 观感层；布置仍 glTF。
- MVP：Polycam/Postshot → PLY → SOG/SPZ → Spark 或 SuperSplat。
- 勿商用 INRIA 参考实现。
- 单图/4D 不进展陈主路径。
- 未点头不进产品代码。
