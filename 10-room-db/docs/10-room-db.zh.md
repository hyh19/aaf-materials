# 10. Room 数据库

到目前为止，你已经有了一个很棒的应用程序，可以在互联网上搜索食谱。但是还没有书签或杂货功能。当你去商店时，你会希望有一份这些食谱所需的配料清单。难道你想在商店里还要再次搜索才能获取这些信息吗？

## SQLite

持久化数据的最佳方式之一是使用数据库。Android 提供了对 **SQLite** 数据库系统的访问。这让你可以插入、读取、更新和移除持久化在磁盘上的结构化数据。

在本章中，你将学习如何使用 **Room** 库。

在本章结束时，你将了解：

+ 如何插入、获取和移除食谱或配料。
+ 如何使用仓库模式（Repository pattern）提供执行这些操作的通用方法。

### Room

Room 是谷歌在 2018 年创建的库，它在 Android 提供的内置 SQLite 数据库之上添加了一层封装，使其更易于使用。

在深入代码之前，了解 Room 的三个基本组件很重要：

2. **数据库**：这是与底层 SQLite 数据库交互的主要接口。这个组件维护一个或多个**数据访问对象**（DAOs）并使用数据库使用的所有**实体**列表进行注解。数据库类继承自 `RoomDatabase` 并使用 `@Database` 注解。

4. **实体**：这代表存储在数据库中的单个数据类型。Room 为每个实体在数据库中创建一个表，表中的行代表单个实体项。

    实体被定义为使用 `@Entity` 注解的简单类。除非你使用 `@Ignore` 注解，否则所有实体类属性都会自动定义为数据库中的字段。你应该使用 `@PrimaryKey` 注解将至少一个实体属性指定为主键。

6. **DAO**：数据访问对象是 Room 的英雄。这里是你定义访问数据库的接口的地方。DAO 应该是你的应用程序中直接与数据库交谈的唯一部分。数据库类必须包含至少一个返回使用 DAO 注解的接口的抽象方法。

下图说明了这三个组件：

<svg width="600" height="682" viewBox="0 0 600 682" fill="none" xmlns="http://www.w3.org/2000/svg">
<g id="pb_room_architecture">
<g id="Arrow">
<path id="Line" d="M253 44V93H210.5H168" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
<circle id="Circle" cx="6" cy="6" r="5" transform="matrix(-1 0 0 1 259 38)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_2">
<path id="Line_2" d="M168 93L168 137" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip" d="M174.345 135.805L168.175 143.805C167.971 144.069 167.571 144.064 167.375 143.794L161.545 135.794C161.304 135.464 161.54 135 161.949 135L173.949 135C174.364 135 174.598 135.477 174.345 135.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
<g id="Arrow_3">
<path id="Line_3" d="M131 383L131 212" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round" stroke-dasharray="2 4"></path>
<path id="Tip_2" d="M124.655 212.195L130.825 204.195C131.029 203.931 131.429 203.936 131.625 204.206L137.455 212.206C137.696 212.536 137.46 213 137.051 213L125.051 213C124.636 213 124.402 212.523 124.655 212.195Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
<g id="Arrow_4">
<path id="Line_4" d="M364 47L364 177" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_3" d="M370.345 175.805L364.175 183.805C363.971 184.069 363.571 184.064 363.375 183.794L357.545 175.794C357.304 175.464 357.54 175 357.949 175L369.949 175C370.364 175 370.598 175.477 370.345 175.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_2" cx="364" cy="41" r="5" transform="rotate(90 364 41)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_5">
<path id="Line_5" d="M266 428L266 506" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_4" d="M272.345 504.805L266.175 512.805C265.971 513.069 265.571 513.064 265.375 512.794L259.545 504.794C259.304 504.464 259.54 504 259.949 504L271.949 504C272.364 504 272.598 504.477 272.345 504.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_3" cx="266" cy="422" r="5" transform="rotate(90 266 422)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Shape">
<rect x="377" y="146" width="117" height="27" rx="8" fill="white"></rect>
<rect x="377" y="146" width="117" height="27" rx="8" stroke="#333333" stroke-width="2"></rect>
<text id="Get/Set Properties" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="388.449" y="163.625">Get/Set Properties</tspan></text>
</g>
<g id="Shape_2">
<rect x="144" y="246" width="96" height="38" rx="8" fill="white"></rect>
<rect x="144" y="246" width="96" height="38" rx="8" stroke="#333333" stroke-width="2"></rect>
<text id="Read/Write Entities" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="163.136" y="263.625">Read/Write
</tspan><tspan x="172.847" y="274.625">Entities</tspan></text>
</g>
<g id="Shape_3">
<rect x="231" y="105" width="96" height="38" rx="8" fill="white"></rect>
<rect x="231" y="105" width="96" height="38" rx="8" stroke="#333333" stroke-width="2"></rect>
<text id="Get DAOs from Database" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="241.773" y="122.625">Get DAOs from
</tspan><tspan x="255.168" y="133.625">Database</tspan></text>
</g>
<g id="ShapeLightBlue">
<rect x="109" y="11" width="307" height="48" rx="13" fill="#A6D9E2"></rect>
<rect x="109" y="11" width="307" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Main Application Code" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="180.484" y="41">Main Application Code</tspan></text>
</g>
<g id="ShapeYellow">
<rect x="109" y="380" width="307" height="48" rx="13" fill="#FFD46F"></rect>
<rect x="109" y="380" width="307" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Database" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="227.836" y="410">Database</tspan></text>
</g>
<g id="Arrow_6">
<path id="Line_6" d="M208 174L253.5 174L253.5 230.5L253.5 287" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
<path id="Tip_5" d="M246.655 285.805L252.825 293.805C253.029 294.069 253.429 294.064 253.625 293.794L259.455 285.794C259.696 285.464 259.46 285 259.051 285L247.051 285C246.636 285 246.402 285.477 246.655 285.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_4" cx="6" cy="6" r="5" transform="matrix(-4.37114e-08 1 1 4.37114e-08 202 168)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="ShapeGreen">
<rect x="114" y="147" width="109" height="48" fill="#D6E18D"></rect>
<rect x="114" y="147" width="109" height="48" stroke="#333333" stroke-width="2"></rect>
<text id="Label" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="151.034" y="176.25">Label</tspan></text>
</g>
<g id="ShapeGreen_2">
<rect x="109" y="152" width="109" height="48" fill="#D6E18D"></rect>
<rect x="109" y="152" width="109" height="48" stroke="#333333" stroke-width="2"></rect>
<text id="DAOs" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="145.959" y="181.25">DAOs</tspan></text>
</g>
<g id="ShapePink">
<rect x="322" y="187" width="94" height="48" fill="#F2BCD7"></rect>
<rect x="322" y="187" width="94" height="48" stroke="#333333" stroke-width="2"></rect>
<text id="Label_2" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="351.534" y="216.25">Label</tspan></text>
</g>
<g id="ShapePink_2">
<rect x="315" y="192" width="96" height="48" fill="#F2BCD7"></rect>
<rect x="315" y="192" width="96" height="48" stroke="#333333" stroke-width="2"></rect>
<text id="Entities" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="338.623" y="221.25">Entities</tspan></text>
</g>
<g id="ShapePink_3">
<rect x="206" y="297" width="94" height="48" fill="#F2BCD7"></rect>
<rect x="206" y="297" width="94" height="48" stroke="#333333" stroke-width="2"></rect>
<text id="Label_3" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="235.534" y="326.25">Label</tspan></text>
</g>
<g id="ShapePink_4">
<rect x="199" y="302" width="96" height="48" fill="#F2BCD7"></rect>
<rect x="199" y="302" width="96" height="48" stroke="#333333" stroke-width="2"></rect>
<text id="Entities_2" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="222.623" y="331.25">Entities</tspan></text>
</g>
<g id="Device">
<g id="ShapePattern">
<rect x="189" y="671" width="155" height="155" rx="77.5" transform="rotate(-90 189 671)" fill="white"></rect>
<rect x="189" y="671" width="155" height="155" rx="77.5" transform="rotate(-90 189 671)" stroke="#333333" stroke-width="2"></rect>
<g id="Pattern">
<path id="Line_7" d="M276.5 670.303V516.697" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
<path id="Line_8" d="M272.5 671.151V516.697" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
<path id="Line_9" d="M268.5 672V515" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
<path id="Line_10" d="M264.5 672V515" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
<path id="Line_11" d="M260.5 671.151V515.849" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
<path id="Line_12" d="M256.5 670.303V516.697" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
</g>
</g>
<g id="ShapeInner">
<rect x="205" y="570" width="123" height="43" rx="13" fill="white"></rect>
<rect x="205" y="570" width="123" height="43" rx="13" stroke="#333333" stroke-width="2"></rect>
<g id="ShapeInnerForeYellow">
<rect x="210" y="575" width="113" height="33" rx="9" fill="#FFD46F"></rect>
<rect x="210" y="575" width="113" height="33" rx="9" stroke="#333333" stroke-width="2"></rect>
<text id="Label_4" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="13" font-weight="500" letter-spacing="0em"><tspan x="217.274" y="596.375">SQLite Database</tspan></text>
</g>
</g>
</g>
</g>
</svg>

#### Room 和 Android 架构组件

Room 是一组名为 **Android 架构组件**的更大库的一部分。其他组件包括：

+ **生命周期管理**：提供几个类来帮助构建生命周期感知对象。
+ **LiveData**：保存可以观察变化的数据并尊重生命周期。
+ **ViewModel**：管理与视图相关的数据，而不与配置更改绑定。这是 UI 视图和应用程序其余部分之间的桥梁。

现在不要担心这些组件的细节；你将在构建应用程序时更详细地介绍它们。

#### 应用架构

在创建第一个 Room 类之前，你必须组织应用程序以实现清晰的架构。你将沿着以下几条线将应用程序分为不同的责任区域：

+ 数据访问和持久化（Room）。
+ 数据模型（Model）。
+ 数据抽象（Repository）。
+ 业务/领域逻辑（ViewModel）。
+ 用户界面（Activity/Fragment）。

一个关键目标是确保这些层之间的通信只朝一个方向流动。这导致了松散耦合的架构，易于修改而没有副作用。

架构看起来是这样的：

<svg width="600" height="610" viewBox="0 0 600 610" fill="none" xmlns="http://www.w3.org/2000/svg">
<g id="db_architecture">
<g id="Arrow">
<path id="Line" d="M236 53L236 109" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip" d="M242.345 107.805L236.175 115.805C235.971 116.069 235.571 116.064 235.375 115.794L229.545 107.794C229.304 107.464 229.54 107 229.949 107L241.949 107C242.364 107 242.598 107.477 242.345 107.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle" cx="236" cy="47" r="5" transform="rotate(90 236 47)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_2">
<path id="Line_2" d="M300 161L300 217" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_2" d="M306.345 215.805L300.175 223.805C299.971 224.069 299.571 224.064 299.375 223.794L293.545 215.794C293.304 215.464 293.54 215 293.949 215L305.949 215C306.364 215 306.598 215.477 306.345 215.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_2" cx="300" cy="155" r="5" transform="rotate(90 300 155)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_3">
<path id="Line_3" d="M468 161L468 541" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_3" d="M474.345 539.805L468.175 547.805C467.971 548.069 467.571 548.064 467.375 547.794L461.545 539.794C461.304 539.464 461.54 539 461.949 539L473.949 539C474.364 539 474.598 539.477 474.345 539.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_3" cx="468" cy="155" r="5" transform="rotate(90 468 155)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_4">
<path id="Line_4" d="M282 269L282 325" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_4" d="M288.345 323.805L282.175 331.805C281.971 332.069 281.571 332.064 281.375 331.794L275.545 323.794C275.304 323.464 275.54 323 275.949 323L287.949 323C288.364 323 288.598 323.477 288.345 323.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_4" cx="282" cy="263" r="5" transform="rotate(90 282 263)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_5">
<path id="Line_5" d="M426 269L426 541" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_5" d="M432.345 539.805L426.175 547.805C425.971 548.069 425.571 548.064 425.375 547.794L419.545 539.794C419.304 539.464 419.54 539 419.949 539L431.949 539C432.364 539 432.598 539.477 432.345 539.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_5" cx="426" cy="263" r="5" transform="rotate(90 426 263)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_6">
<path id="Line_6" d="M257 379L257 435" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_6" d="M263.345 433.805L257.175 441.805C256.971 442.069 256.571 442.064 256.375 441.794L250.545 433.794C250.304 433.464 250.54 433 250.949 433L262.949 433C263.364 433 263.598 433.477 263.345 433.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_6" cx="257" cy="373" r="5" transform="rotate(90 257 373)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_7">
<path id="Line_7" d="M384 377L384 541" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_7" d="M390.345 539.805L384.175 547.805C383.971 548.069 383.571 548.064 383.375 547.794L377.545 539.794C377.304 539.464 377.54 539 377.949 539L389.949 539C390.364 539 390.598 539.477 390.345 539.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_7" cx="384" cy="371" r="5" transform="rotate(90 384 371)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Arrow_8">
<path id="Line_8" d="M236 485L236 541" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_8" d="M242.345 539.805L236.175 547.805C235.971 548.069 235.571 548.064 235.375 547.794L229.545 539.794C229.304 539.464 229.54 539 229.949 539L241.949 539C242.364 539 242.598 539.477 242.345 539.805Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle_8" cx="236" cy="479" r="5" transform="rotate(90 236 479)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="ShapeLightBlue">
<rect x="110" y="227" width="342" height="48" rx="13" fill="#A6D9E2"></rect>
<rect x="110" y="227" width="342" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Repository" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="241.461" y="257">Repository</tspan></text>
</g>
<g id="ShapePink">
<rect x="110" y="335" width="293" height="48" rx="13" fill="#F2BCD7"></rect>
<rect x="110" y="335" width="293" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Data Access" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="211.805" y="365">Data Access</tspan></text>
</g>
<g id="ShapePurple">
<rect x="110" y="443" width="250" height="48" rx="13" fill="#D3BDDB"></rect>
<rect x="110" y="443" width="250" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Persistence" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="192.211" y="473">Persistence</tspan></text>
</g>
<g id="ShapePurple_2">
<rect x="110" y="551" width="379" height="48" rx="13" fill="#D3BDDB"></rect>
<rect x="110" y="551" width="379" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Data Model" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="257.977" y="581">Data Model</tspan></text>
</g>
<g id="ShapeYellow">
<rect x="110" y="11" width="250" height="48" rx="13" fill="#FFD46F"></rect>
<rect x="110" y="11" width="250" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="UI (Compose)" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="184.633" y="41">UI (Compose)</tspan></text>
</g>
<g id="ShapeGreen">
<rect x="110" y="119" width="379" height="48" rx="13" fill="#D6E18D"></rect>
<rect x="110" y="119" width="379" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="ViewModel" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="259.07" y="149">ViewModel</tspan></text>
</g>
</g>
</svg>

箭头代表通信和可见性线。请注意，UI 层完全独立于除 ViewModel 之外的所有其他层。ViewModel 层对 UI 层一无所知。

当你构建应用程序的其余部分时，你不会在坚持严格遵循上图所示的通信流程方面做出妥协。有时需要做更多的工作才能严格遵守此模式，但对于较大的应用程序来说，付出的努力是值得的。即使对于一个小型应用程序，你也可以立即认识到一些好处：

+ 在 Room 中存储数据的方式可以完全替换，影响最小。唯一受影响的层是**持久化**层及其直接父级，即**数据访问**层。

+ 你可以替换 UI 层，而不会让任何其他层知道。

+ 你可以在没有任何活动 UI 运行的情况下轻松测试所有层。

#### 开发方法

将架构想象成一个多层蛋糕。你有没有见过有人一次吃一层蛋糕？这有点奇怪！同样，你不会一次构建一层应用程序。你会一次取一片。每一片可能会穿过所有层，你慢慢构建最终产品。

<svg width="600" height="425" viewBox="0 0 600 425" fill="none" xmlns="http://www.w3.org/2000/svg">
<g id="db_architecture_cake">
<g id="ShapeYellow">
<rect x="11" y="45" width="192" height="48" fill="#FFD46F"></rect>
<rect x="11" y="45" width="192" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeGreen">
<rect x="11" y="93" width="192" height="48" fill="#D6E18D"></rect>
<rect x="11" y="93" width="192" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue">
<rect x="11" y="141" width="192" height="48" fill="#A6D9E2"></rect>
<rect x="11" y="141" width="192" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeRed">
<rect x="11" y="285" width="192" height="48" fill="#F7B39C"></rect>
<rect x="11" y="285" width="192" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapePurple">
<rect x="11" y="237" width="192" height="48" fill="#D3BDDB"></rect>
<rect x="11" y="237" width="192" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapePink">
<rect x="11" y="189" width="192" height="48" fill="#F2BCD7"></rect>
<rect x="11" y="189" width="192" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeYellow_2">
<rect x="342" y="45" width="247" height="48" fill="#FFD46F"></rect>
<rect x="342" y="45" width="247" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeGreen_2">
<rect x="342" y="93" width="247" height="48" fill="#D6E18D"></rect>
<rect x="342" y="93" width="247" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_2">
<rect x="342" y="141" width="247" height="48" fill="#A6D9E2"></rect>
<rect x="342" y="141" width="247" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeRed_2">
<rect x="342" y="285" width="247" height="48" fill="#F7B39C"></rect>
<rect x="342" y="285" width="247" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapePurple_2">
<rect x="342" y="237" width="247" height="48" fill="#D3BDDB"></rect>
<rect x="342" y="237" width="247" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapePink_2">
<rect x="342" y="189" width="247" height="48" fill="#F2BCD7"></rect>
<rect x="342" y="189" width="247" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeYellow_3">
<rect x="175" y="69" width="138" height="48" fill="#FFD46F"></rect>
<rect x="175" y="69" width="138" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeGreen_3">
<rect x="175" y="117" width="138" height="48" fill="#D6E18D"></rect>
<rect x="175" y="117" width="138" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_3">
<rect x="175" y="165" width="138" height="48" fill="#A6D9E2"></rect>
<rect x="175" y="165" width="138" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeRed_3">
<rect x="175" y="309" width="138" height="48" fill="#F7B39C"></rect>
<rect x="175" y="309" width="138" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapePurple_3">
<rect x="175" y="261" width="138" height="48" fill="#D3BDDB"></rect>
<rect x="175" y="261" width="138" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapePink_3">
<rect x="175" y="213" width="138" height="48" fill="#F2BCD7"></rect>
<rect x="175" y="213" width="138" height="48" stroke="#333333" stroke-width="2"></rect>
</g>
<text id="UI (Compose)" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="33.4287" y="74.25">UI (Compose)</tspan></text>
<text id="ViewModel" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="33.124" y="122.25">ViewModel</tspan></text>
<text id="Repository" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="33.4033" y="170.25">Repository</tspan></text>
<text id="Data Access" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="33.3916" y="218.25">Data Access</tspan></text>
<text id="Persistence (Room)" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="33.1426" y="266.25">Persistence (Room)</tspan></text>
<text id="Data Model" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="33.167" y="314.25">Data Model</tspan></text>
<g id="Arrow">
<path id="Line" d="M174.986 68.1215L200.671 47.4027" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round" stroke-dasharray="2 4"></path>
</g>
<text id="The Architecture Cake" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="427.305" y="23">The Architecture Cake</tspan></text>
<g id="Arrow_2">
<path id="Line_2" d="M361 400L244.5 400L244.5 383.5L244.5 367" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
<path id="Tip" d="M251.345 368.195L245.175 360.195C244.971 359.931 244.571 359.936 244.375 360.206L238.545 368.206C238.304 368.536 238.54 369 238.949 369L250.949 369C251.364 369 251.598 368.523 251.345 368.195Z" fill="white" stroke="#333333" stroke-width="2"></path>
<circle id="Circle" cx="6" cy="6" r="5" transform="matrix(1.19249e-08 -1 -1 -1.19249e-08 367 406)" fill="white" stroke="#333333" stroke-width="2"></circle>
</g>
<g id="Shape">
<rect x="341" y="385" width="140" height="29" rx="8" fill="white"></rect>
<rect x="341" y="385" width="140" height="29" rx="8" stroke="#333333" stroke-width="2"></rect>
<text id="One slice at a time" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="13" font-weight="500" letter-spacing="0em"><tspan x="356.245" y="404.375">One slice at a time</tspan></text>
</g>
<g id="Arrow_3">
<path id="Line_3" d="M312.986 68.1215L339.449 46.7748" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round" stroke-dasharray="2 4"></path>
</g>
<g id="Arrow_4">
<path id="Line_4" d="M314.29 357.032L340.241 335.065" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round" stroke-dasharray="2 4"></path>
</g>
</g>
</svg>

以下是你将使用的目录：

+ **data/database**：数据访问和持久化。你将在这里保存 **Room 数据库**和 **DAO** 对象。
+ **data/models**：模型对象。这包括所有 **Room 实体**类。
+ **ui**：用户界面。所有视图和视图控制逻辑都属于这里。
+ **viewmodels**：业务/领域逻辑。这包含驱动用户界面和应用程序逻辑的 ViewModel 类。

#### 添加 Room 库

如果你正在跟随前几章的应用程序，请打开并继续使用它进行本章的内容。如果没有，请找到本章的 **projects** 文件夹并在 Android Studio 中打开 **starter**。

如果你使用的是 starter 项目，请打开 **SpoonacularService.kt** 文件，并使用你在 [https://www.spoonacular.com](https://www.spoonacular.com/) 创建的账户中的 API 密钥更新 **apiKey**。

打开 **libs.versions.toml** 文件添加 Room 库。在 **versions** 部分的末尾，添加：

```
room="2.5.2"
```

然后，在 **libraries** 部分的末尾，添加：

```
# Room
room = { module= "androidx.room:room-ktx", version.ref="room" }
room-runtime ={ module= "androidx.room:room-runtime", version.ref="room" }
room-compiler = { module = "androidx.room:room-compiler", version.ref="room" }
```

最后，打开应用模块的 **build.gradle.kts** 文件并添加：

```
id("kotlin-parcelize")
```

作为 **plugins** 部分的最后一行。使用 `@Parcelize` 注解时，此插件会为 `Parcelable` 类型自动生成代码。

然后，在 **dependencies** 部分，添加：

```
// Room
implementation(libs.room)
implementation(libs.room.runtime)
ksp (libs.room.compiler)
```

并执行 Gradle 同步。

## Room 类

现在，你已经准备好添加 Room 所需的基本类。这包括实体、DAO 和数据库。在后台，Room 会根据你的类结构创建一个带有表和列定义的 SQLite 数据库。

对于 Room，将数据库命名为：`recipe_database`，模型类命名为：`RecipeDb` 和 `IngredientDb`。下图将帮助你可视化 Room 用于将类转换为底层数据库的过程：

<svg width="600" height="630" viewBox="0 0 600 630" fill="none" xmlns="http://www.w3.org/2000/svg">
<g id="db_to_database">
<g id="ShapeYellow">
<rect x="589" y="499" width="305" height="98" rx="49" transform="rotate(-180 589 499)" fill="#FFD46F"></rect>
<rect x="589" y="499" width="305" height="98" rx="49" transform="rotate(-180 589 499)" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeYellow_2">
<rect x="11" y="11" width="145" height="195" rx="13" fill="#FFD46F"></rect>
<rect x="11" y="11" width="145" height="195" rx="13" stroke="#333333" stroke-width="2"></rect>
</g>
<text id="Database" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="53.6689" y="42.25">Database</tspan></text>
<g id="ShapeGreen">
<rect x="19" y="102" width="129" height="77" fill="#D6E18D"></rect>
<rect x="19" y="102" width="129" height="77" stroke="#333333" stroke-width="2"></rect>
<text id="RecipeDao IngredientDao" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="12" font-weight="500" letter-spacing="0em"><tspan x="54.0156" y="139">RecipeDao
</tspan><tspan x="43.7793" y="151">IngredientDao</tspan></text>
</g>
<g id="ShapeLightBlue">
<rect x="19" y="64" width="129" height="38" fill="#A6D9E2"></rect>
<rect x="19" y="64" width="129" height="38" stroke="#333333" stroke-width="2"></rect>
<text id="RecipeDatabase" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="12" font-weight="500" letter-spacing="0em"><tspan x="38.793" y="86.5">RecipeDatabase</tspan></text>
</g>
<g id="ShapeLightBlue_2">
<rect x="284" y="148" width="305" height="303" fill="#FFD46F"></rect>
<rect x="284" y="148" width="305" height="303" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeYellow_3">
<rect x="1" y="240" width="155" height="379" rx="13" fill="#FFD46F"></rect>
<rect x="1" y="240" width="155" height="379" rx="13" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeGreen_2">
<rect x="18" y="319" width="131" height="129" fill="#D6E18D"></rect>
<rect x="18" y="319" width="131" height="129" stroke="#333333" stroke-width="2"></rect>
<text id="id:Int title: String image: String? summary: String instructions:String? sourceUrl: String preparationMinutes: Int cookingMinutes: Int readyInMinutes: Int servings: Int" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="23" y="338.125">id:Int
</tspan><tspan x="23" y="349.125">title: String
</tspan><tspan x="23" y="360.125">image: String?
</tspan><tspan x="23" y="371.125">summary: String
</tspan><tspan x="23" y="382.125">instructions:String?
</tspan><tspan x="23" y="393.125">sourceUrl: String
</tspan><tspan x="23" y="404.125">preparationMinutes: Int
</tspan><tspan x="23" y="415.125">cookingMinutes: Int
</tspan><tspan x="23" y="426.125">readyInMinutes: Int
</tspan><tspan x="23" y="437.125">servings: Int
</tspan></text>
</g>
<text id="Model Entities" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="14" font-weight="500" letter-spacing="0em"><tspan x="38.2881" y="267.25">Model Entities</tspan></text>
<g id="ShapeLightBlue_3">
<rect x="18" y="281" width="131" height="38" fill="#A6D9E2"></rect>
<rect x="18" y="281" width="131" height="38" stroke="#333333" stroke-width="2"></rect>
<text id="RecipeDb" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="12" font-weight="500" letter-spacing="0em"><tspan x="57.1445" y="303.5">RecipeDb</tspan></text>
</g>
<g id="ShapeGreen_3">
<rect x="18" y="495" width="131" height="110" fill="#D6E18D"></rect>
<rect x="18" y="495" width="131" height="110" stroke="#333333" stroke-width="2"></rect>
<text id="id:Int recipeId: Int? name: String aisle:String? image: String? original: String amount: Double unit: String" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="23" y="513.625">id:Int
</tspan><tspan x="23" y="524.625">recipeId: Int?
</tspan><tspan x="23" y="535.625">name: String
</tspan><tspan x="23" y="546.625">aisle:String?
</tspan><tspan x="23" y="557.625">image: String?
</tspan><tspan x="23" y="568.625">original: String
</tspan><tspan x="23" y="579.625">amount: Double
</tspan><tspan x="23" y="590.625">unit: String</tspan></text>
</g>
<g id="ShapeLightBlue_4">
<rect x="18" y="457" width="131" height="38" fill="#A6D9E2"></rect>
<rect x="18" y="457" width="131" height="38" stroke="#333333" stroke-width="2"></rect>
<text id="IngredientDb" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="12" font-weight="500" letter-spacing="0em"><tspan x="46.9082" y="479.5">IngredientDb</tspan></text>
</g>
<g id="ShapeYellow_4">
<rect x="285" y="97" width="304" height="95" rx="47.5" fill="#FFD46F"></rect>
<rect x="285" y="97" width="304" height="95" rx="47.5" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="Device">
<g id="ShapePattern">
<rect x="589" y="192" width="305" height="95" rx="47.5" transform="rotate(-180 589 192)" fill="white"></rect>
<rect x="589" y="192" width="305" height="95" rx="47.5" transform="rotate(-180 589 192)" stroke="#333333" stroke-width="2"></rect>
<g id="Pattern">
<path id="Line" d="M590 142.5L283 142.5" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
<path id="Line_2" d="M590 146.5L283 146.5" stroke="#333333" stroke-width="2" stroke-miterlimit="16"></path>
</g>
</g>
</g>
<g id="ShapeInner">
<rect x="365" y="120" width="139" height="47" rx="13" fill="white"></rect>
<rect x="365" y="120" width="139" height="47" rx="13" stroke="#333333" stroke-width="2"></rect>
<g id="ShapeInnerForeYellow">
<rect x="370" y="125" width="129" height="37" rx="9" fill="#FFD46F"></rect>
<rect x="370" y="125" width="129" height="37" rx="9" stroke="#333333" stroke-width="2"></rect>
<text id="Label" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="13" font-weight="500" letter-spacing="0em"><tspan x="385.433" y="148.375">recipe_database</tspan></text>
</g>
</g>
<g id="ShapePink">
<rect x="290.191" y="223.626" width="35.3021" height="22.9317" fill="#F2BCD7"></rect>
<rect x="290.191" y="223.626" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="id" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="303.131" y="239.217">id</tspan></text>
</g>
<g id="ShapePink_2">
<rect x="325.628" y="223.626" width="38.0997" height="22.9317" fill="#F2BCD7"></rect>
<rect x="325.628" y="223.626" width="38.0997" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="title" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="334.58" y="239.217">title</tspan></text>
</g>
<g id="ShapePink_3">
<rect x="363.862" y="223.626" width="43.695" height="22.9317" fill="#F2BCD7"></rect>
<rect x="363.862" y="223.626" width="43.695" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="image" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="370.407" y="239.217">image</tspan></text>
</g>
<g id="ShapePink_4">
<rect x="407.692" y="223.626" width="54.8856" height="22.9317" fill="#F2BCD7"></rect>
<rect x="407.692" y="223.626" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="summary" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="411.529" y="239.217">summary</tspan></text>
</g>
<g id="ShapePink_5">
<rect x="462.713" y="223.626" width="66.0762" height="22.9317" fill="#F2BCD7"></rect>
<rect x="462.713" y="223.626" width="66.0762" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="instructions" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="465.839" y="239.217">instructions</tspan></text>
</g>
<g id="ShapePink_6">
<rect x="528.924" y="223.626" width="54.8856" height="22.9317" fill="#F2BCD7"></rect>
<rect x="528.924" y="223.626" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="sourceUrl" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="532.003" y="239.217">sourceUrl</tspan></text>
</g>
<g id="ShapeLightBlue_5">
<rect x="290.191" y="246.656" width="35.3021" height="22.9317" fill="#A6D9E2"></rect>
<rect x="290.191" y="246.656" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_6">
<rect x="325.628" y="246.656" width="38.0997" height="22.9317" fill="#A6D9E2"></rect>
<rect x="325.628" y="246.656" width="38.0997" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_7">
<rect x="363.862" y="246.656" width="43.695" height="22.9317" fill="#A6D9E2"></rect>
<rect x="363.862" y="246.656" width="43.695" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_8">
<rect x="407.692" y="246.656" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="407.692" y="246.656" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_9">
<rect x="462.713" y="246.656" width="66.0762" height="22.9317" fill="#A6D9E2"></rect>
<rect x="462.713" y="246.656" width="66.0762" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_10">
<rect x="528.924" y="246.656" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="528.924" y="246.656" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_11">
<rect x="290.191" y="269.686" width="35.3021" height="22.9317" fill="#A6D9E2"></rect>
<rect x="290.191" y="269.686" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_12">
<rect x="325.628" y="269.686" width="38.0997" height="22.9317" fill="#A6D9E2"></rect>
<rect x="325.628" y="269.686" width="38.0997" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_13">
<rect x="363.862" y="269.686" width="43.695" height="22.9317" fill="#A6D9E2"></rect>
<rect x="363.862" y="269.686" width="43.695" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_14">
<rect x="407.692" y="269.686" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="407.692" y="269.686" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_15">
<rect x="462.713" y="269.686" width="66.0762" height="22.9317" fill="#A6D9E2"></rect>
<rect x="462.713" y="269.686" width="66.0762" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_16">
<rect x="528.924" y="269.686" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="528.924" y="269.686" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_17">
<rect x="290.191" y="292.715" width="35.3021" height="22.9317" fill="#A6D9E2"></rect>
<rect x="290.191" y="292.715" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_18">
<rect x="325.628" y="292.715" width="38.0997" height="22.9317" fill="#A6D9E2"></rect>
<rect x="325.628" y="292.715" width="38.0997" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_19">
<rect x="363.862" y="292.715" width="43.695" height="22.9317" fill="#A6D9E2"></rect>
<rect x="363.862" y="292.715" width="43.695" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_20">
<rect x="407.692" y="292.715" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="407.692" y="292.715" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_21">
<rect x="462.713" y="292.715" width="66.0762" height="22.9317" fill="#A6D9E2"></rect>
<rect x="462.713" y="292.715" width="66.0762" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_22">
<rect x="528.924" y="292.715" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="528.924" y="292.715" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeGreen_4">
<rect x="290.191" y="201" width="293.619" height="22.9317" fill="#D6E18D"></rect>
<rect x="290.191" y="201" width="293.619" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="Recipe Table" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="13" font-weight="500" letter-spacing="0em"><tspan x="399.022" y="217.341">Recipe Table</tspan></text>
</g>
<g id="Group 2">
<g id="ShapePink_7">
<rect x="290.191" y="345.705" width="35.3021" height="22.9317" fill="#F2BCD7"></rect>
<rect x="290.191" y="345.705" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="id_2" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="303.131" y="361.296">id</tspan></text>
</g>
<g id="ShapePink_8">
<rect x="325.628" y="345.432" width="53.0205" height="22.9317" fill="#F2BCD7"></rect>
<rect x="325.628" y="345.432" width="53.0205" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="recipeId" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="330.944" y="361.022">recipeId</tspan></text>
</g>
<g id="ShapePink_9">
<rect x="377.851" y="345.432" width="52.088" height="22.9317" fill="#F2BCD7"></rect>
<rect x="377.851" y="345.432" width="52.088" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="name" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="389.779" y="361.022">name</tspan></text>
</g>
<g id="ShapePink_10">
<rect x="429.141" y="345.432" width="48.3578" height="22.9317" fill="#F2BCD7"></rect>
<rect x="429.141" y="345.432" width="48.3578" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="aisle" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="441.503" y="361.022">aisle</tspan></text>
</g>
<g id="ShapePink_11">
<rect x="477.634" y="345.432" width="51.1554" height="22.9317" fill="#F2BCD7"></rect>
<rect x="477.634" y="345.432" width="51.1554" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="image_2" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="487.909" y="361.022">image</tspan></text>
</g>
<g id="ShapePink_12">
<rect x="528.924" y="345.705" width="54.8856" height="22.9317" fill="#F2BCD7"></rect>
<rect x="528.924" y="345.705" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="original" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="11" font-weight="500" letter-spacing="0em"><tspan x="537.514" y="361.296">original</tspan></text>
</g>
<g id="ShapeLightBlue_23">
<rect x="290.191" y="368.735" width="35.3021" height="22.9317" fill="#A6D9E2"></rect>
<rect x="290.191" y="368.735" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_24">
<rect x="325.628" y="368.644" width="53.0205" height="22.9317" fill="#A6D9E2"></rect>
<rect x="325.628" y="368.644" width="53.0205" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_25">
<rect x="377.851" y="368.644" width="52.088" height="22.9317" fill="#A6D9E2"></rect>
<rect x="377.851" y="368.644" width="52.088" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_26">
<rect x="429.141" y="368.644" width="48.3578" height="22.9317" fill="#A6D9E2"></rect>
<rect x="429.141" y="368.644" width="48.3578" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_27">
<rect x="477.634" y="368.644" width="51.1554" height="22.9317" fill="#A6D9E2"></rect>
<rect x="477.634" y="368.644" width="51.1554" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_28">
<rect x="528.924" y="368.735" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="528.924" y="368.735" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_29">
<rect x="290.191" y="391.765" width="35.3021" height="22.9317" fill="#A6D9E2"></rect>
<rect x="290.191" y="391.765" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_30">
<rect x="325.628" y="391.856" width="53.0205" height="22.9317" fill="#A6D9E2"></rect>
<rect x="325.628" y="391.856" width="53.0205" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_31">
<rect x="377.851" y="391.856" width="52.088" height="22.9317" fill="#A6D9E2"></rect>
<rect x="377.851" y="391.856" width="52.088" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_32">
<rect x="429.141" y="391.856" width="48.3578" height="22.9317" fill="#A6D9E2"></rect>
<rect x="429.141" y="391.856" width="48.3578" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_33">
<rect x="477.634" y="391.856" width="51.1554" height="22.9317" fill="#A6D9E2"></rect>
<rect x="477.634" y="391.856" width="51.1554" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_34">
<rect x="528.924" y="391.765" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="528.924" y="391.765" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_35">
<rect x="290.191" y="414.794" width="35.3021" height="22.9317" fill="#A6D9E2"></rect>
<rect x="290.191" y="414.794" width="35.3021" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_36">
<rect x="325.628" y="415.068" width="53.0205" height="22.9317" fill="#A6D9E2"></rect>
<rect x="325.628" y="415.068" width="53.0205" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_37">
<rect x="377.851" y="415.068" width="52.088" height="22.9317" fill="#A6D9E2"></rect>
<rect x="377.851" y="415.068" width="52.088" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_38">
<rect x="429.141" y="415.068" width="48.3578" height="22.9317" fill="#A6D9E2"></rect>
<rect x="429.141" y="415.068" width="48.3578" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_39">
<rect x="477.634" y="415.068" width="51.1554" height="22.9317" fill="#A6D9E2"></rect>
<rect x="477.634" y="415.068" width="51.1554" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeLightBlue_40">
<rect x="528.924" y="414.794" width="54.8856" height="22.9317" fill="#A6D9E2"></rect>
<rect x="528.924" y="414.794" width="54.8856" height="22.9317" stroke="#333333" stroke-width="2"></rect>
</g>
<g id="ShapeGreen_5">
<rect x="290.191" y="323.079" width="293.619" height="22.9317" fill="#D6E18D"></rect>
<rect x="290.191" y="323.079" width="293.619" height="22.9317" stroke="#333333" stroke-width="2"></rect>
<text id="Ingredient Table" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="13" font-weight="500" letter-spacing="0em"><tspan x="387.933" y="339.42">Ingredient Table</tspan></text>
</g>
</g>
<g id="ShapePurple">
<rect x="187" y="144" width="66" height="48" rx="13" fill="#D3BDDB"></rect>
<rect x="187" y="144" width="66" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Room" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="198.789" y="174">Room</tspan></text>
</g>
<g id="ShapePurple_2">
<rect x="187" y="247" width="66" height="48" rx="13" fill="#D3BDDB"></rect>
<rect x="187" y="247" width="66" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Room_2" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="198.789" y="277">Room</tspan></text>
</g>
<g id="ShapePurple_3">
<rect x="187" y="488" width="66" height="48" rx="13" fill="#D3BDDB"></rect>
<rect x="187" y="488" width="66" height="48" rx="13" stroke="#333333" stroke-width="2"></rect>
<text id="Room_3" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="198.789" y="518">Room</tspan></text>
</g>
<g id="Group 4">
<g id="Arrow">
<path id="Line_3" d="M147 127L164 127L164 147.5L164 168" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
</g>
<g id="Arrow_2">
<path id="Line_4" d="M164 168L177 168" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip" d="M175.805 161.655L183.805 167.825C184.069 168.029 184.064 168.429 183.794 168.625L175.794 174.455C175.464 174.696 175 174.46 175 174.051V162.051C175 161.636 175.477 161.402 175.805 161.655Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
</g>
<g id="Group 6">
<g id="Arrow_3">
<path id="Line_5" d="M149 301L166 301L166 286L166 271" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
</g>
<g id="Arrow_4">
<path id="Line_6" d="M166 271L177 271" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_2" d="M175.805 277.345L183.805 271.175C184.069 270.971 184.064 270.571 183.794 270.375L175.794 264.545C175.464 264.304 175 264.54 175 264.949V276.949C175 277.364 175.477 277.598 175.805 277.345Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
</g>
<g id="Group 8">
<g id="Arrow_5">
<path id="Line_7" d="M254 271L271 271L271 242L271 213" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
</g>
<g id="Arrow_6">
<path id="Line_8" d="M271 213L282 213" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_3" d="M280.805 219.345L288.805 213.175C289.069 212.971 289.064 212.571 288.794 212.375L280.794 206.545C280.464 206.304 280 206.54 280 206.949V218.949C280 219.364 280.477 219.598 280.805 219.345Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
</g>
<g id="Group 9">
<g id="Arrow_7">
<path id="Line_9" d="M254 513L265 513L265 418L265 323" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
</g>
<g id="Arrow_8">
<path id="Line_10" d="M265 323L276 323" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_4" d="M273.805 329.345L281.805 323.175C282.069 322.971 282.064 322.571 281.794 322.375L273.794 316.545C273.464 316.304 273 316.54 273 316.949V328.949C273 329.364 273.477 329.598 273.805 329.345Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
</g>
<g id="Group 7">
<g id="Arrow_9">
<path id="Line_11" d="M149 544L166 544L166 529L166 514" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
</g>
<g id="Arrow_10">
<path id="Line_12" d="M166 514L177 514" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_5" d="M175.805 520.345L183.805 514.175C184.069 513.971 184.064 513.571 183.794 513.375L175.794 507.545C175.464 507.304 175 507.54 175 507.949V519.949C175 520.364 175.477 520.598 175.805 520.345Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
</g>
<g id="Group 5">
<g id="Arrow_11">
<path id="Line_13" d="M222 143L222 64L329.5 64L437 64" stroke="#333333" stroke-width="2" stroke-miterlimit="1.11658" stroke-linejoin="round"></path>
</g>
<g id="Arrow_12">
<path id="Line_14" d="M436 65.0001L436 87.0001" stroke="#333333" stroke-width="2" stroke-miterlimit="16" stroke-linecap="round"></path>
<path id="Tip_6" d="M442.345 85.8054L436.175 93.8052C435.971 94.0692 435.571 94.0637 435.375 93.7943L429.545 85.7945C429.304 85.4641 429.54 85.0001 429.949 85.0001L441.949 85.0001C442.364 85.0001 442.598 85.4767 442.345 85.8054Z" fill="white" stroke="#333333" stroke-width="2"></path>
</g>
</g>
<text id="Room Database Creation Process" fill="#333333" xml:space="preserve" style="white-space: pre" font-family="IBM Plex Sans" font-size="16" font-weight="500" letter-spacing="0em"><tspan x="309.039" y="30">Room Database Creation Process</tspan></text>
</g>
</svg>

### 实体

Recipe Finder 需要两种实体类型来存储食谱：`RecipeDb` 和 `IngredientDb`。

#### RecipeDb

在 **data** 包中创建一个名为 **database** 的包。在这个包中，创建一个名为 **RecipeDb.kt** 的 Kotlin 文件，并将内容替换为以下内容：

```
import android.os.Parcelable
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.parcelize.Parcelize

// 1
@Parcelize
// 2
@Entity(tableName = "recipes")
// 3
data class RecipeDb(
  // 4
  @PrimaryKey(autoGenerate = false)
  // 5
  @ColumnInfo(name = "id")
  var id: Int,
  @ColumnInfo(name = "title")
  var title: String,
  @ColumnInfo(name = "image")
  var image: String?,
  @ColumnInfo(name = "summary")
  var summary: String = "",
  @ColumnInfo(name = "instructions")
  var instructions: String? = "",
  @ColumnInfo(name = "sourceUrl")
  var sourceUrl: String = "",
  @ColumnInfo(name = "preparationMinutes")
  var preparationMinutes: Int = 0,
  @ColumnInfo(name = "cookingMinutes")
  var cookingMinutes: Int = 0,
  @ColumnInfo(name = "readyInMinutes")
  var readyInMinutes: Int = 0,
  @ColumnInfo(name = "servings")
  var servings: Int = 0,
) : Parcelable
```

以下是上面代码的解释：

2. Kotlin 使用 `@Parcelize` 注解来生成类的 `Parcelable` 实现。`Parcelable` 是一个接口，用于序列化和反序列化对象。这对于在 Android 应用中的活动、片段或其他组件之间传输数据很有用。

4. `@Entity` 注解告诉 Room 这是一个数据库实体类。

    > **注意**：虽然在这个例子中没有使用，但你可以对 Entity 注解应用几个属性。
    >
    > `foreignKeys()`：外键约束列表。
    >
    > `indices()`：要包含在表中的索引列表。
    >
    > `primaryKeys()`：主键列名称列表。如果使用 `PrimaryKey` 注解，则不需要。
    >
    > `tableName()`：在数据库中使用的表名。默认为类名。

6. `RecipeDb` 类的主构造函数使用定义了默认值的所有属性的参数来定义。定义默认值允许你使用部分属性列表构造食谱。

    > **注意**：Room 在定义表字段时查找构造函数上的参数和类属性。在这种情况下，你只使用属性来定义表字段。

8. 你使用 `@PrimaryKey` 注解定义了 `id` 属性。每个实体类必须至少有一个这样的注解。`autoGenerate` 属性自动告诉 Room 为这个字段生成递增的数字。

    在数据库术语中，这被认为是一个代理键或合成键，为每个食谱记录提供唯一标识符。

10. 你定义了其余的字段，并带有默认值。

#### IngredientDb

在 **data/database** 包中创建一个名为 **IngredientDb.kt** 的 Kotlin 文件，并将内容替换为以下内容：

```
import android.os.Parcelable
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.parcelize.Parcelize

@Parcelize
@Entity(tableName = "ingredients")
data class IngredientDb(
  @PrimaryKey(autoGenerate = false)
  @ColumnInfo(name = "id")
  var id: Int,
  @ColumnInfo(name = "recipeId")
  var recipeId: Int?,
  @ColumnInfo(name = "name")
  var name: String,
  @ColumnInfo(name = "aisle")
  var aisle: String? = "",
  @ColumnInfo(name = "image")
  var image: String? = "",
  @ColumnInfo(name = "original")
  var original: String = "",
  @ColumnInfo(name = "amount")
  var amount: Double = 0.0,
  @ColumnInfo(name = "unit")
  var unit: String = "",
) : Parcelable
```

这与 `RecipeDb` 类类似。

### DAOs

接下来，你将定义数据访问对象，用于从数据库读取和写入数据。

在 **data/database** 包中创建一个名为 **RecipeDao.kt** 的 Kotlin 文件，并将内容替换为以下内容：

```
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update

// 1
@Dao
interface RecipeDao {
  // 2
  @Insert(onConflict = OnConflictStrategy.IGNORE)
  suspend fun addRecipe(recipe: RecipeDb)

  // 3
  @Query("SELECT * FROM recipes WHERE id = :id")
  suspend fun findRecipeById(id: Int): RecipeDb

  // 4
  @Query("SELECT * FROM recipes")
  suspend fun getAllRecipes(): List<RecipeDb>

  // 5
  @Update
  suspend fun updateRecipeDetails(recipe: RecipeDb)

  // 6
  @Delete
  suspend fun deleteRecipe(recipe: RecipeDb)

  @Query("DELETE FROM recipes WHERE id = :recipeId")
  suspend fun deleteRecipeById(recipeId: Int)
}
```

`RecipeDao` 定义了传统上称为 **CRUD** 数据库操作。CRUD 操作包括：

+ *C*：创建。在数据库中创建新对象。
+ *R*：读取。从数据库读取对象。
+ *U*：更新。更新数据库中的对象。
+ *D*：删除。从数据库中删除对象。

对食谱数据的所有访问都将通过这个类。你可以随意命名这些方法，但真正的力量在于注解。`@Query`、`@Insert`、`@Update` 和 `@Delete` 注解为 Room 提供了有价值的信息。Room 使用这些信息生成代码，自动将数据实体转换为数据库行，反之亦然。

这个类引入了几个新概念：

2. `@Dao` 注解告诉 Room 这是一个**数据访问对象**。DAO 类必须是接口或抽象类。Room 根据你定义的方法定义在运行时创建具体类。

4. 你使用 `@Insert` 注解定义了 `addRecipe()`。这将单个 `RecipeDb` 对象保存到数据库，并返回与新食谱关联的新主键 ID。`@Insert` 注解的 `onConflict` 属性定义了当存在具有相同主键的现有记录时会发生什么。

    > **注意**：要了解有关冲突选项的更多信息，请参阅此页面：[https://developer.android.com/reference/androidx/room/OnConflictStrategy](https://developer.android.com/reference/androidx/room/OnConflictStrategy)。有关每种冲突策略的底层详细信息以及 SQLite 如何定义它们的更多信息，请参阅：[https://sqlite.org/lang\_conflict.html](https://sqlite.org/lang_conflict.html)。

6. 这个方法返回单个 `RecipeDb` 对象。在这里，你使用 `@Query` 注解告诉 Room 如何检索单个食谱。此方法基于 `id` 加载 `RecipeDb` 对象。要执行数据库查询，Room 接受传递到你的方法中的参数，并替换查询中匹配的 `:?` 字符串，其中 `?` 匹配方法上的参数名称。在这种情况下，它用传递给 `findRecipeById()` 的 `id` 参数的值替换 `:id`。

8. `getAllRecipes()` 使用 `@Query` 注解来定义一个 SQL 语句，用于从数据库读取所有食谱并将它们作为 `List` 的 `Recipes` 返回。

    > **注意**：SQL，即结构化查询语言，是一种众所周知的方法，用于处理关系型数据库，如 SQLite。构建应用程序时，你不需要了解太多 SQL。如果你想了解更多关于 SQL 的信息，特别是 SQLite 使用的语法，请阅读 [https://sqlite.org/lang.html](https://sqlite.org/lang.html)。

10. 你使用 `@Update` 注解定义了 `updateRecipeDetails()`。这使用传入的 `recipe` 参数更新数据库中的单个食谱。

12. 最后，你使用 `@Delete` 注解定义了 `deleteRecipe()` 和使用带有自定义 `DELETE` 语句的 `@Query` 注解定义了 `deleteRecipeById()`。这基于传入的 `RecipeDb` 对象或 `recipeId` 删除现有食谱。

在 **data/database** 包中创建一个名为 **IngredientDao.kt** 的 Kotlin 文件，并将内容替换为以下内容：

```
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update

@Dao
interface IngredientDao {
  @Insert(onConflict = OnConflictStrategy.IGNORE)
  suspend fun addIngredient(ingredientDb: IngredientDb)

  @Query("SELECT * FROM ingredients WHERE id = :id")
  suspend fun findIngredientById(id: Int): IngredientDb

  @Query("SELECT * FROM ingredients WHERE recipeId = :id")
  suspend fun findIngredientsByRecipe(id: Int): List<IngredientDb>

  @Query("SELECT * FROM ingredients")
  suspend fun getAllIngredients(): List<IngredientDb>

  @Update
  suspend fun updateIngredientDetails(ingredientDb: IngredientDb)

  @Delete
  suspend fun deleteIngredient(ingredientDb: IngredientDb)
}
```

这与 `RecipeDao` 类类似。

### 数据库

完成 Room 类所需的最后一个部分是数据库。

在 **data/database** 包中创建一个名为 **RecipeDatabase.kt** 的 Kotlin 文件，并将内容替换为以下内容：

```
import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

// 1
@Database(entities = [RecipeDb::class, IngredientDb::class], version = 1, exportSchema = false)
abstract class RecipeDatabase : RoomDatabase() {
  // 2
  abstract fun recipeDao(): RecipeDao
  abstract fun ingredientDao(): IngredientDao

  // 3
  companion object {
      /*volatile 变量的值永远不会被缓存，所有的写入和读取都将直接在主内存中完成。
      这有助于确保 INSTANCE 的值始终是最新的，并且对所有执行线程都是相同的。
      这意味着一个线程对 INSTANCE 所做的更改对所有其他线程立即可见。*/
      @Volatile
      // 4
      private var INSTANCE: RecipeDatabase? = null

      // 5
      fun getInstance(context: Context): RecipeDatabase {
        // 一次只有一个执行线程可以进入此代码块
        synchronized(this) {
          var instance = INSTANCE

         // 6
          if (instance == null) {
            instance = Room.databaseBuilder(
                context.applicationContext,
                RecipeDatabase::class.java,
                "recipe_database"
            ).fallbackToDestructiveMigration()
                .build()

            INSTANCE = instance
          }
          // 7
          return instance
        }
      }
  }
}
```

这段代码的工作原理如下：

2. `@Database` 注解向 Room 标识一个 `Database` 类。`entities` 是 `@Database` 注解中的必需属性，它定义了数据库使用的所有实体的数组。这个数据库将存储两个实体。

    Room 要求你的数据库类是抽象的并继承自 `RoomDatabase`。

4. 抽象方法 `recipeDao` 和 `ingredientDao` 被定义为返回 DAO 接口。请注意，你可以拥有任意多的 DAO。你将其声明为抽象的，因为 Room 根据你之前定义的接口为你实现 `RecipeDao` 和 `IngredientDao` 类。

    这就是 `Database` 类所需的全部内容。其余代码让你将 `Database` 接口对象用作单例。谷歌推荐这样做，因为创建新的 `Database` 对象可能很昂贵。

6. 在 `RecipeDatabase` 上定义一个 `companion object`。

8. 在伴生对象上定义唯一的 `instance` 变量。

10. 定义 `getInstance()` 接受一个 `Context` 并返回单个 `RecipeDatabase` 实例。

12. 如果这是第一次调用 `getInstance`，则创建单个 `RecipeDatabase` 实例。`Room.databaseBuilder()` 基于抽象的 `RecipeDatabase` 类创建一个 Room 数据库。

14. 返回 `RecipeDatabase` 实例。

> **注意**：现在你已经定义了数据库，你可以测试 Room 的一个很棒的功能。它在编译时验证 @Query 注解中的 SQL。
>
> 如果 SQL 语法有错误，例如引用了不存在的表名，它会给你一个错误。如果你的方法的返回类型与 SQL 语句的返回类型不匹配，它也会警告你。
>
> 通过将 **RecipeDao.kt** 中的一个 @Query 字符串中的 `recipes` 更改为 `recipe` 来测试这一点。注意 Android Studio 会将其标记为错误。如果你尝试构建项目，它会产生一个编译错误，内容为："查询存在问题：\[SQLITE\_ERROR\] SQL 错误或缺少数据库（没有这样的表：recipe）"。
>
> 如果你曾经在 Room 可用之前使用过 Android SQLite 数据库，你会意识到这有多么有用。Room 提供了一个安全网，防止 SQL 语句中的常见拼写错误。

#### 创建仓库

你的基本 Room 类已经准备就绪。但是你将在 Room 和应用程序其余代码之间添加另一层抽象。这样做可以轻松更改应用程序数据的存储方式和位置。这个抽象层将使用**仓库**模式提供。仓库是一个通用的数据存储，可以管理多个数据源，但向应用程序的其余部分公开统一的接口。

你将创建一个名为 `RecipeRepository` 的单一仓库类来管理你的食谱书签。这个类将在内部使用 `RecipeDatabase` 中的 `RecipeDao` 和 `IngredientDao` 来访问底层的食谱和配料。它将定义一些用于保存和加载的基本方法。

在 **data** 包中创建一个名为 **RecipeRepository.kt** 的 Kotlin 文件，并将内容替换为以下内容：

```
import com.kodeco.recipefinder.data.database.IngredientDao
import com.kodeco.recipefinder.data.database.IngredientDb
import com.kodeco.recipefinder.data.database.RecipeDao
import com.kodeco.recipefinder.data.database.RecipeDatabase
import com.kodeco.recipefinder.data.database.RecipeDb

// 1
class RecipeRepository(recipeDatabase: RecipeDatabase) {
  // 2
  private val recipeDao: RecipeDao = recipeDatabase.recipeDao()
  private val ingredientDao: IngredientDao = recipeDatabase.ingredientDao()

  // 3
  suspend fun findAllRecipes(): List<RecipeDb> {
    return recipeDao.getAllRecipes()
  }

  suspend fun findBookmarkById(id: Int): RecipeDb {
    return recipeDao.findRecipeById(id)
  }

  suspend fun findRecipeById(id: Int): RecipeDb {
    return recipeDao.findRecipeById(id)
  }

  suspend fun findAllIngredients(): List<IngredientDb> {
    return ingredientDao.getAllIngredients()
  }

  suspend fun findRecipeIngredients(recipeId: Int): List<IngredientDb> {
    return ingredientDao.findIngredientsByRecipe(recipeId)
  }

 // 4
 suspend fun insertRecipe(recipe: RecipeDb) {
    recipeDao.addRecipe(recipe)
  }

  suspend fun insertIngredients(ingredients: List<IngredientDb>) {
    ingredients.forEach {
        ingredientDao.addIngredient(it)
    }
  }

  // 5
  suspend fun deleteRecipe(recipe: RecipeDb) {
    recipeDao.deleteRecipe(recipe)
  }

  suspend fun deleteRecipeById(recipeId: Int) {
    recipeDao.deleteRecipeById(recipeId)
  }

  suspend fun deleteIngredient(ingredient: IngredientDb) {
    ingredientDao.deleteIngredient(ingredient)
  }

  suspend fun deleteIngredients(ingredients: List<IngredientDb>) {
    ingredients.forEach {
        ingredientDao.deleteIngredient(it)
    }
  }

  suspend fun deleteRecipeIngredients(recipeId: Int) {
    val ingredients = findRecipeIngredients(recipeId)
    ingredients.forEach {
        ingredientDao.deleteIngredient(it)
    }
  }
}
```

以下是代码的详细解析：

2. 定义带有一个构造函数的 `RecipeRepository` 类，该构造函数传入 `RecipeDatabase`。

4. `RecipeRepository` 使用这两个属性作为其数据源。第一个是 `RecipeDao`，第二个是来自 `IngredientDao` 的 `DAO` 对象。

6. `findAllRecipes()` 返回所有食谱的列表。

8. 创建 `insertRecipe()` 来添加单个食谱。

10. 添加 `deleteRecipe()` 来删除特定食谱。

在构建 ViewModel 时，你将看到如何详细使用这个类。

### ViewModels

现在你已经构建了仓库，你将在 ViewModel 中使用它，这是一个放置它的完美位置。

首先，你需要取消注释一些方法，用于在仓库和 UI 之间转换食谱。打开 **data/Conversions.kt** 并取消注释代码。

接下来，打开 **RecipeViewModel.kt**。找到 `// TODO: Add Repository` 并用以下内容更新 ViewModel 的构造函数：

```
class RecipeViewModel(
  private val prefs: Prefs,
  private val repository: RecipeRepository,
) : ViewModel() {
```

确保添加 `RecipeRepository` 的导入。

现在，找到 `// TODO: get Bookmarks`。请注意，你将这些称为书签，即使它们是食谱。这是因为你在给食谱添加书签。用以下内容替换该方法：

```
suspend fun getBookmarks() {
  withContext(Dispatchers.IO) {
    val allRecipes = repository.findAllRecipes()
    _bookmarksState.value = recipeDbsToRecipes(allRecipes).toMutableList()
  }
}
```

你必须导入一些类。这将在 IO 协程调度器上运行调用，确保它在后台运行。使用传递给 ViewModel 构造函数的仓库，查找所有已添加书签的食谱并更新书签状态（这通知 UI 变化）。对 `getIngredients()` 方法做同样的事情：

```
suspend fun getIngredients() {
  withContext(Dispatchers.IO) {
    val allIngredients = repository.findAllIngredients()
    _ingredientsState.value = ingredientDbsToIngredients(allIngredients).toMutableList()
  }
}
```

确保添加 `ingredientDbsToIngredients` 的导入。这获取所有配料，将它们转换为 UI 模型并设置配料状态列表。

要获取书签，将 `// TODO: Get Bookmark` 替换为：

```
suspend fun getBookmark(bookmarkId: Int) {
  withContext(Dispatchers.IO) {
    val recipe = repository.findRecipeById(bookmarkId)
    val ingredients = repository.findRecipeIngredients(bookmarkId)
    _recipeState.value =
        recipeDbToRecipeInformation(recipe, ingredientDbsToExtendedIngredients(ingredients))
  }
}
```

添加 `recipeDbToRecipeInformation` 和 `ingredientDbsToExtendedIngredients` 的导入。这找到食谱及其配料并创建食谱信息。

要保存单个书签，将 `bookmarkRecipe()` 方法替换为：

```
suspend fun bookmarkRecipe(recipe: RecipeInformationResponse) {
  withContext(Dispatchers.IO) {
    repository.insertRecipe(recipeInformationToRecipeDb(recipe))
    repository.insertIngredients(
      extendedIngredientsToIngredientDbs(
        recipe.id,
        recipe.extendedIngredients
      )
    )
  }
}
```

添加 `recipeInformationToRecipeDb` 和 `extendedIngredientsToIngredientDbs` 的导入。在这里，你需要插入食谱及其配料。

要删除食谱，找到第一个 `// TODO: Delete Bookmark` 并将该方法替换为：

```
suspend fun deleteBookmark(recipe: Recipe) {
  withContext(Dispatchers.IO) {
    // 1
    val recipeDb = recipeToDb(recipe)
    // 2
    repository.deleteRecipe(recipeDb)
    repository.deleteRecipeIngredients(recipe.id)
    // 3
    val localList = _bookmarksState.value.toMutableList()
    // 4
    localList.remove(recipe)
    _bookmarksState.value = localList
  }
}
```

确保添加 `recipeToDb` 的导入。以下是你所做的：

2. 将 UI 食谱模型转换为 DB 食谱模型。
4. 从仓库中删除食谱和配料。
6. 将当前书签列表转换为可变列表。
8. 从列表中删除食谱并更新书签状态中的列表。

找到下一个 `// TODO: Delete Bookmark`。这是删除书签的另一种方式。如果你只有食谱 ID 而不是食谱本身，这个方法就能派上用场。将该方法替换为：

```
suspend fun deleteBookmark(recipeId: Int) {
  withContext(Dispatchers.IO) {
    repository.deleteRecipeById(recipeId)
    repository.deleteRecipeIngredients(recipeId)
    val localList = _bookmarksState.value.toMutableList()
    localList.removeIf {
      it.id == recipeId
    }
    _bookmarksState.value = localList
  }
}
```

这与你之前编写的代码类似。主要区别在于你使用的是需要 `recipeId` 的仓库函数。

这完成了 ViewModel。

### 实例化仓库

现在你已经设置了仓库的使用，是时候创建它了。就像 `RecipeApp` 中创建 `Prefs` 实例的方式一样，你将创建一个新的 `RecipeRepository` 实例。首先打开 `RecipeApp` 并找到第一个 `// TODO: Add Repository` 注释。用以下内容替换该注释：

```
lateinit var repository: RecipeRepository
```

添加 `RecipeRepository` 的导入。现在，找到第二个 `// TODO: Add Repository` 注释并用以下内容替换：

```
repository = RecipeRepository(
  Room.databaseBuilder(
    this,
    RecipeDatabase::class.java,
    "Recipes"
  ).build()
)
```

这使用 Room 创建了仓库的新实例。你需要导入 `RecipeDatabase` 和 `Room`。

### 本地仓库提供者

很多类使用仓库。你如何向 UI 中的所有可组合项提供该仓库呢？通过使用本地提供者概念。这是一种向其他可组合项提供类的方法。你将在更高级别的可组合项中创建类，并使用本地提供者来提供该实例。打开 **MainActivity.kt** 并在 `LocalNavigatorProvider` 全局变量之后添加：

```
val LocalRepositoryProvider =
    compositionLocalOf<RecipeRepository> { error("No repository provided") }
```

这创建了一个作为本地提供者的全局变量。

你需要导入：

```
import com.kodeco.recipefinder.data.RecipeRepository
```

找到 `// TODO: Add LocalRepositoryProvider` 并用以下内容替换：

```
LocalRepositoryProvider provides (application as RecipeApp).repository,
```

添加 `LocalRepositoryProvider` 和 `RecipeApp` 的导入。

这使用在 `RecipeApp` 中创建和存储的 `RecipeRepository` 并将其添加到你的本地仓库提供者中。

### 书签

现在是时候更新 **ui/recipes/ShowBookmarks.kt** 文件了。找到 `// TODO: Provide current item` 注释并用以下内容替换其下方的方法调用：

```
viewModel.deleteBookmark(currentItem)
```

### 食谱详情

要更新的最后一个文件是 **ui/RecipeDetails.kt**。找到第一个 `// TODO: Add Repository` 并用以下内容替换：

```
val repository = LocalRepositoryProvider.current
```

并导入 `LocalRepositoryProvider`。然后，用以下内容替换 `RecipeViewModel` 工厂实例化：

```
RecipeViewModel(prefs, repository)
```

找到 `// TODO: Provide recipe ID` 并用以下内容替换其下方的方法调用：

```
viewModel.getBookmark(databaseRecipeId)
```

再往下，找到下一个 `// TODO: Provide recipe ID` 并用以下内容替换：

```
 viewModel.deleteBookmark(recipe.id)
```

以便使用给定的食谱 ID 删除书签。找到下一个 `// TODO: Provide recipe` 并用以下内容替换：

```
viewModel.bookmarkRecipe(recipe)
```

这是相反的操作，添加书签。

### 更新 ViewModel 引用

因为你更新了 `RecipeViewModel` 以接收仓库，所以必须修复 `GroceryList` 和 `RecipeList` 可组合函数中的实例化。要修复这个问题，请打开以下文件

+ **ui/recipes/RecipeList.kt**
+ **ui/groceries/GroceryList.kt**

找到 `// TODO: Add Repository` 并添加：

```
val repository = LocalRepositoryProvider.current
```

并导入 `LocalRepositoryProvider`。

这获取当前仓库。现在，用以下内容更新 `RecipeViewModel` 实例化：

```
RecipeViewModel(prefs, repository)
```

### 更新预览

在构建和运行更改之前的最后一个要求是更新预览可组合项。几个预览使用 `RecipeViewModel` 现在需要一个仓库。在以下每个类中：

+ **ui/recipes/ChipRow.kt**
+ **ui/recipes/SearchRow.kt**
+ **ui/recipes/ShowBookmarks.kt**
+ **ui/recipes/ShowRecipeList.kt**

在每个文件底部的相关预览方法中查找 `// TODO: Add Repository` 注释，并用以下内容替换：

```
val repository = LocalRepositoryProvider.current
```

添加 `LocalRepositoryProvider` 的导入。然后，用以下内容更新 `RecipeViewModel` 实例化：

```
RecipeViewModel(prefs, repository)
```

最后，你已经准备好测试并确保你的应用程序正常工作。运行应用程序并搜索你喜欢的食物。点击图像进入详情，然后点击书签图标：

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original.png)

当你点击书签图标时，你会返回到列表。点击顶部的书签按钮：

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(1).png)

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(2).png)

要删除书签，向左或向右滑动。要查看食谱，点击卡片。恭喜！你现在有一个功能齐全的食谱查找器应用程序，可以保存书签食谱。

### 杂货

如果你点击底部的杂货按钮，你会看到没有杂货。要修复这个问题，打开 **ui/groceries/GroceryList.kt**。找到 `// TODO: Get Ingredients` 并用以下内容替换：

```
scope.launch {
  recipeViewModel.getIngredients()
}
```

这检索当前的配料列表。如果你查看上面的代码：

```
scope.launch {
  recipeViewModel.ingredientsState.collect { ingredients ->
    groceryListViewModel.setIngredients(ingredients)
  }
}
```

你可以看到它正在监听配料列表的变化。重新启动应用程序并确保配料出现在杂货页面上。

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(3).png)

#### 替代方案

你可以使用 SQLite 和围绕它的类，但创建和维护数据库需要一些工作。以下是一些替代方案：

+ **GreenDAO**（[https://greenrobot.org/greendao/](https://greenrobot.org/greendao/)）：易于使用的开源 ORM。
+ **Realm**（[https://realm.io/](https://realm.io/)）：一个快速的数据库，需要低级 C++ 代码，不使用 SQLite。
+ **Firebase Realtime Database**（[https://firebase.google.com/docs/database/](https://firebase.google.com/docs/database/)）：托管在云中的 NoSQL 数据库。付费。
+ **Apollo/GraphQL**（[https://graphql.org/](https://graphql.org/)）：仅在你有使用 GraphQL 的远程数据库时使用。GraphQL 是由 Facebook 创建的一种更灵活的新格式，比 REST 格式更灵活。许多公司正在转向这种格式。

## 要点

+ Room 是创建数据库和保存数据的好方法。
+ 创建数据库并存储食谱很容易。

## 下一步

在本章中，你学习了如何在数据库中存储数据。

要了解有关 Room 的更多信息，请访问：[https://developer.android.com/training/data-storage/room](https://developer.android.com/training/data-storage/room)。

在下一章中，你将学习高级存储技术。我们再见！
