专科疾病智能风险分级 
1.1 把一句话拆成矩阵
原任务："针对骨科、胸外、肝胆、泌尿等专科，提供疾病严重程度分级、手术风险预判、预后评估建议。"
拆开就是 3 × 4 = 12 个输出：
	骨科	胸外	肝胆	泌尿
严重程度分级	骨折分型	结节分类	肿瘤分期	风险分层
手术风险预判	麻醉+合并症	肺功能+麻醉	肝功能+切除范围	麻醉+肾功能
预后评估建议	生存/功能	分期生存	分期生存	风险组生存
关键洞察：12 个输出长得完全不一样，但结构是同构的——都是"若干变量 → 分值 → 等级 → 依据"。这就是整个框架的地基。
```
2. 技术栈
2.1 选型
层	选型	版本	为什么
语言	Python	3.11	三个人都会一点
界面	Streamlit	≥1.32	一个脚本出网页，不用写 HTML/JS
图表	Plotly	≥5.18	雷达图、仪表盘开箱即用，可交互
文档	python-docx	≥1.1	报告导出，API 简单
测试	标准库 `assert`	—	不引入 pytest，降低门槛
版本	Git + GitHub	—	三人协作的最低要求
总共三个第三方包。 安装 1 分钟，环境问题几乎为零——这是这个任务相对影像类任务最大的优势，一定要利用好。
---
3. 工程结构
3.1 目录
```
risk-demo/
├── app.py                  # 入口（只有这一个页面）
├── engine/                #工程核心
│   ├── __init__.py        #把目录标记成python包
│   ├── scorer.py           # 规则解释器 + 通用层
│   ├── registry.py         # 注册表 + evaluate()
│   └── rules/                 #四个专科的知识
│       ├── __init__.py
│       ├── ortho.py        # 骨科：SPEC + prognosis
│       ├── thoracic.py
│       ├── hepato.py
│       └── urology.py
├── ui/                       #渲染
│   ├── __init__.py
│   ├── forms.py  #	自动渲染出输入框、下拉框、勾选框，返回一个 dict
│   ├── cards.py  #	三张结果卡（严重程度、手术风险、预后）+ 可展开的依据列表 + 免责声明
│   └── charts.py  #雷达图（四维归一化到 0–1）、可选的仪表盘和分值条形图
├── report.py     #输出层
├── samples/     #演示的保底
│   ├── ortho_01.json
│   ├── thoracic_01.json
│   ├── hepato_01.json
│   └── urology_01.json
├── check.py                # 算例校验脚本（不用 pytest）
├── docs/
│   └── 评分依据.md
├── out/  运行时产物：临时图片、生成的 docx
├── .streamlit/config.toml 主题色、上传大小限制、端口	可选，但设了主题色显得更完整
├── requirements.txt  依赖文本
└── README.md
```
4. 三人分工
4.1 角色
成员	角色	负责文件	
A	界面与输出	`app.py`、`ui/*`、`report.py`	
B	引擎与两个专科	`engine/scorer.py`、`engine/registry.py`、`ortho.py`、`thoracic.py`	
C	两个专科与材料	`hepato.py`、`urology.py`、`samples/`、`docs/评分依据.md`、PPT	
C 兼任项目经理：管 main 分支、每天合并、文献核对、演示脚本。
