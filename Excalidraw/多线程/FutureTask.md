---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Text Elements
run()调用 ^K6eQ3Yri

任务执行状态为New？ ^Vs5bq5gW

基于CAS修改任务执行线程为当前线程 ^UnZxWzy6

修改成功？ ^AmE4RpVO

false ^lQpg6fL3

false ^nMySckpy

调用call方法执行 ^e29lam92

执行异常？ ^sb0rmmRa

记录执行结果 ^aoF3VM2U

记录异常结果 ^EJTurcMa

false ^UARlYq9J

true ^rIaH69NZ

基于CAS替换任务状态为COMPLETING ^IUQSbQnv

替换成功？ ^GdQllVQs

修改任务状态为NORMAL ^aZtLVDU1

基于CAS替换任务状态为COMPLETING ^c914oc4R

修改任务状态为EXCEPTIONAL ^yKgBnkGb

开始唤醒等待结果的线程 ^sPy1gUpM

获取当前等待栈中的头节点直至为空 ^oho7mx0g

若该节点线程不为空，唤醒该线程 ^zd3VOJAR

获取该节点的next节点 ^adq0cLAN

利用CAS操作将其将其替换为新的栈顶节点 ^FArjZvq6

唤醒结束 ^d77UtmCp

节点为空？ ^mVuUfApF

自旋 ^NTvg7Q5p

false ^jBzy7z86

true ^Jv0IbGhS

替换成功？ ^T5r7R9oR

结束 ^AElEUsEH

当前任务状态为INTERRUPTING？ ^Vvhjt22S

让出当前线程执行权 ^Ns8bSZRe

false ^a1dXJYmM

true ^ji8Uhr8l

false ^gxoBKnld

false ^qPv7TiEr

结束 ^w62p7tP1

true ^zML2yvQj

get()调用 ^p2O955cT

任务执行状态为New或COMPLETING？ ^AusEUnF9

返回结果 ^UtomQQ6i

开始自旋等待
任务完成 ^kgXqSavI

判断任务
当前状态 ^K350siRl

若等待节点线程不为null，将其置为null ^Tfawvd1v

构建等待节点并基于CAS加入等待队列中 ^AM5nj69Y

移除等待节点 ^fMmcpUdM

开始阻塞
等待唤醒 ^SxceXMks

任务已正常或异常结束 ^s8c4PjMX

未完成，构建等待节点 ^eb879XU4

已超时或已被中断 ^KYVg5yeB

未完成，需阻塞等待 ^26l9fjxB

自旋 ^zKde4LdN

将当前节点线程置为null ^jvppgQQi

从等待栈中遍历，开始移除被取消的节点 ^X6glx9Ay

队列头节点已取消等待？ ^JHJRt7Zc

基于CAS操作移除该头节点 ^AlARBGTG

当前节点等待线程不为空？ ^lNoJKoPb

移除该节点 ^OAqhfD8K

true ^zmGJzcEa

false ^ZKGNosKV

false
自旋 ^VFtWO9BK

自旋 ^JsJeb6jv

返回结果 ^tacezu1X

true ^hob9cdvw

false ^LbLJk4wl

被取消的节点可能由以下原因产生：
等待节点刚构建完成并加入等待队列中，但尚未阻塞时发现任务已完成，
此时无需阻塞直接返回结果，因此等待线程为空 ^JMBMNq0b

true ^RTX8onvM

%%
# Drawing
```json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/1.9.18",
	"elements": [
		{
			"type": "rectangle",
			"version": 238,
			"versionNonce": 1896325445,
			"isDeleted": false,
			"id": "xN791WNQ9pmCGT1k7u15o",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -247.865234375,
			"y": -291.27103327871555,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 170,
			"height": 100,
			"seed": 1664658251,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "K6eQ3Yri"
				},
				{
					"id": "Yh4CWGYZjKctyKOXBA9u6",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 180,
			"versionNonce": 1596576773,
			"isDeleted": false,
			"id": "K6eQ3Yri",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -205.0452117919922,
			"y": -253.77103327871555,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 84.35995483398438,
			"height": 25,
			"seed": 905500715,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "run()调用",
			"rawText": "run()调用",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "xN791WNQ9pmCGT1k7u15o",
			"originalText": "run()调用",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "diamond",
			"version": 1148,
			"versionNonce": 960170853,
			"isDeleted": false,
			"id": "CX5xMl4OrvrYqZzGla1Vs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -262.365234375,
			"y": -120.28096910891497,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 199,
			"height": 172,
			"seed": 2067346245,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Vs5bq5gW"
				},
				{
					"id": "Yh4CWGYZjKctyKOXBA9u6",
					"type": "arrow"
				},
				{
					"id": "xiqDN8tr8xDqGdVK9nthw",
					"type": "arrow"
				},
				{
					"id": "s-PCEma14zuDSG0rJ0nT0",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1451,
			"versionNonce": 1200337125,
			"isDeleted": false,
			"id": "Vs5bq5gW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -204.5252227783203,
			"y": -71.78096910891497,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 83.81997680664062,
			"height": 75,
			"seed": 1806516267,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "任务执行\n状态为Ne\nw？",
			"rawText": "任务执行状态为New？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "CX5xMl4OrvrYqZzGla1Vs",
			"originalText": "任务执行状态为New？",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 572,
			"versionNonce": 1997425733,
			"isDeleted": false,
			"id": "bju_GjPlF7ZYXp1ned-Rc",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -247.865234375,
			"y": 209.6924437058185,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 170,
			"height": 100,
			"seed": 2053828587,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "UnZxWzy6"
				},
				{
					"id": "s-PCEma14zuDSG0rJ0nT0",
					"type": "arrow"
				},
				{
					"id": "sfI4jaFfeOw79bdd5hTjp",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 577,
			"versionNonce": 2054472197,
			"isDeleted": false,
			"id": "UnZxWzy6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -241.94522094726562,
			"y": 222.1924437058185,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.15997314453125,
			"height": 75,
			"seed": 1794882187,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693411327671,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "基于CAS修改任务\n执行线程为当前\n线程",
			"rawText": "基于CAS修改任务执行线程为当前线程",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "bju_GjPlF7ZYXp1ned-Rc",
			"originalText": "基于CAS修改任务执行线程为当前线程",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "diamond",
			"version": 1266,
			"versionNonce": 851935685,
			"isDeleted": false,
			"id": "UWFZlT-1W-8-b3HzUoorH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -244.865234375,
			"y": 371.70494505922875,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 164,
			"height": 126,
			"seed": 1512155429,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AmE4RpVO"
				},
				{
					"id": "sfI4jaFfeOw79bdd5hTjp",
					"type": "arrow"
				},
				{
					"id": "t-Q1Gt6WwGi1-NB1BQrfV",
					"type": "arrow"
				},
				{
					"id": "IEO7moKWZ-21JRbmT-5Qg",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1581,
			"versionNonce": 1534719813,
			"isDeleted": false,
			"id": "AmE4RpVO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -192.865234375,
			"y": 409.70494505922875,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 60,
			"height": 50,
			"seed": 1824210053,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "修改成\n功？",
			"rawText": "修改成功？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "UWFZlT-1W-8-b3HzUoorH",
			"originalText": "修改成功？",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "arrow",
			"version": 478,
			"versionNonce": 275586443,
			"isDeleted": false,
			"id": "Yh4CWGYZjKctyKOXBA9u6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -164.1955108024115,
			"y": -183.27103327871555,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 1.3562476068710225,
			"height": 59.13234240464138,
			"seed": 1484286533,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676284952,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "xN791WNQ9pmCGT1k7u15o",
				"gap": 8,
				"focus": -1.5073133930478994e-16
			},
			"endBinding": {
				"elementId": "CX5xMl4OrvrYqZzGla1Vs",
				"gap": 4.701002935987233,
				"focus": -0.04771339509157502
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1.3562476068710225,
					59.13234240464138
				]
			]
		},
		{
			"type": "arrow",
			"version": 785,
			"versionNonce": 909473099,
			"isDeleted": false,
			"id": "xiqDN8tr8xDqGdVK9nthw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -195.36667876633197,
			"y": 47.09661020485839,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 263.94744472614866,
			"height": 565.7424763967543,
			"seed": 533044587,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "lQpg6fL3"
				}
			],
			"updated": 1693676285048,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CX5xMl4OrvrYqZzGla1Vs",
				"gap": 17.756070879698157,
				"focus": -0.3724091343711138
			},
			"endBinding": {
				"elementId": "r4ct2buhKKsXXFHoClvip",
				"gap": 9.682255956531094,
				"focus": -0.19078236582689895
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-172.66066498366803,
					202.00523318622663
				],
				[
					-263.94744472614866,
					565.7424763967543
				]
			]
		},
		{
			"type": "text",
			"version": 45,
			"versionNonce": 713367909,
			"isDeleted": false,
			"id": "lQpg6fL3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -393.08731842041016,
			"y": 236.60184339108503,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 39244101,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "xiqDN8tr8xDqGdVK9nthw",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 624,
			"versionNonce": 766658219,
			"isDeleted": false,
			"id": "s-PCEma14zuDSG0rJ0nT0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -163.6307944444941,
			"y": 62.578264008328475,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 1.7746503681888726,
			"height": 146.11417969749004,
			"seed": 2104226923,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "zML2yvQj"
				}
			],
			"updated": 1693676284969,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CX5xMl4OrvrYqZzGla1Vs",
				"gap": 10.886185103820367,
				"focus": -0.004129220112683159
			},
			"endBinding": {
				"elementId": "bju_GjPlF7ZYXp1ned-Rc",
				"gap": 1,
				"focus": -0.03690852053549422
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1.7746503681888726,
					146.11417969749004
				]
			]
		},
		{
			"type": "text",
			"version": 32,
			"versionNonce": 1696451621,
			"isDeleted": false,
			"id": "zML2yvQj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -183.9652099609375,
			"y": 124.46842996621018,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 42.199951171875,
			"height": 25,
			"seed": 858207237,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "s-PCEma14zuDSG0rJ0nT0",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 533,
			"versionNonce": 1583356203,
			"isDeleted": false,
			"id": "sfI4jaFfeOw79bdd5hTjp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -164.39799338423737,
			"y": 317.6924437058185,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 1.3420649909165263,
			"height": 50.78408869499191,
			"seed": 1139878027,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676284973,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "bju_GjPlF7ZYXp1ned-Rc",
				"gap": 8,
				"focus": -3.890053627708645e-17
			},
			"endBinding": {
				"elementId": "UWFZlT-1W-8-b3HzUoorH",
				"gap": 4.3115447644954585,
				"focus": -0.05640285706026215
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1.3420649909165263,
					50.78408869499191
				]
			]
		},
		{
			"type": "arrow",
			"version": 993,
			"versionNonce": 1317723787,
			"isDeleted": false,
			"id": "t-Q1Gt6WwGi1-NB1BQrfV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -199.25322521914168,
			"y": 494.51880675997126,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 193.11334037956757,
			"height": 145.81896512982524,
			"seed": 1606086987,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "nMySckpy"
				}
			],
			"updated": 1693676285049,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "UWFZlT-1W-8-b3HzUoorH",
				"gap": 19.64258521300738,
				"focus": -0.5132930607512521
			},
			"endBinding": {
				"elementId": "r4ct2buhKKsXXFHoClvip",
				"gap": 5.177399742953966,
				"focus": 0.001366499669962648
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-193.11334037956757,
					145.81896512982524
				]
			]
		},
		{
			"type": "text",
			"version": 40,
			"versionNonce": 692757061,
			"isDeleted": false,
			"id": "nMySckpy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -320.9849779563075,
			"y": 483.61274193875954,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 1235880299,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "t-Q1Gt6WwGi1-NB1BQrfV",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 637,
			"versionNonce": 447222181,
			"isDeleted": false,
			"id": "5NcHHLHHiA4sKfl4g52Ga",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -247.865234375,
			"y": 581.330359016085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 170,
			"height": 100,
			"seed": 73121157,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "e29lam92"
				},
				{
					"id": "IEO7moKWZ-21JRbmT-5Qg",
					"type": "arrow"
				},
				{
					"id": "5liynzCjkD7sv5v6KTBCE",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 674,
			"versionNonce": 1898502085,
			"isDeleted": false,
			"id": "e29lam92",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -239.81522369384766,
			"y": 618.830359016085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 153.8999786376953,
			"height": 25,
			"seed": 57918693,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "调用call方法执行",
			"rawText": "调用call方法执行",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "5NcHHLHHiA4sKfl4g52Ga",
			"originalText": "调用call方法执行",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "diamond",
			"version": 1340,
			"versionNonce": 69441317,
			"isDeleted": false,
			"id": "2J0ewkOFyfhyrJ1njN8pG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -244.865234375,
			"y": 744.0871949535851,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 164,
			"height": 126,
			"seed": 1292945797,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "sb0rmmRa"
				},
				{
					"id": "5liynzCjkD7sv5v6KTBCE",
					"type": "arrow"
				},
				{
					"id": "FriooeXG-GrTREil29gE8",
					"type": "arrow"
				},
				{
					"id": "i9ksv6doAZq733308n94t",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1672,
			"versionNonce": 2129681573,
			"isDeleted": false,
			"id": "sb0rmmRa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -192.865234375,
			"y": 782.0871949535851,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 60,
			"height": 50,
			"seed": 161960165,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "执行异\n常？",
			"rawText": "执行异常？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "2J0ewkOFyfhyrJ1njN8pG",
			"originalText": "执行异常？",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 727,
			"versionNonce": 2087157765,
			"isDeleted": false,
			"id": "pBsRWwjDsaWHq43yAlHWj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -360.1831822878912,
			"y": 943.7180543285849,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#96f2d7",
			"width": 170,
			"height": 100,
			"seed": 2028819653,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "aoF3VM2U"
				},
				{
					"id": "FriooeXG-GrTREil29gE8",
					"type": "arrow"
				},
				{
					"id": "olgR6bg-75pjYssV4AHs1",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 867,
			"versionNonce": 946758181,
			"isDeleted": false,
			"id": "aoF3VM2U",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -335.1831822878912,
			"y": 981.2180543285849,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 120,
			"height": 25,
			"seed": 2018538533,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "记录执行结果",
			"rawText": "记录执行结果",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "pBsRWwjDsaWHq43yAlHWj",
			"originalText": "记录执行结果",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 785,
			"versionNonce": 856216965,
			"isDeleted": false,
			"id": "ppPVkQPP-xV5bbsCrbzZN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -55.08218488828197,
			"y": 943.7180543285849,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 170,
			"height": 100,
			"seed": 2122654757,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "EJTurcMa"
				},
				{
					"id": "i9ksv6doAZq733308n94t",
					"type": "arrow"
				},
				{
					"id": "NWhTeskKHwHSoO2TJZBmi",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 972,
			"versionNonce": 771598245,
			"isDeleted": false,
			"id": "EJTurcMa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -30.082184888281972,
			"y": 981.2180543285849,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 120,
			"height": 25,
			"seed": 1291278213,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "记录异常结果",
			"rawText": "记录异常结果",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ppPVkQPP-xV5bbsCrbzZN",
			"originalText": "记录异常结果",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 421,
			"versionNonce": 55835883,
			"isDeleted": false,
			"id": "IEO7moKWZ-21JRbmT-5Qg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -159.69606945459782,
			"y": 498.4422476632539,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 1.0683186722435778,
			"height": 74.88811135283112,
			"seed": 907546821,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676284979,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "UWFZlT-1W-8-b3HzUoorH",
				"gap": 2.5154612023321263,
				"focus": -0.04973672420769974
			},
			"endBinding": {
				"elementId": "5NcHHLHHiA4sKfl4g52Ga",
				"gap": 8,
				"focus": 0.01485703852216133
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1.0683186722435778,
					74.88811135283112
				]
			]
		},
		{
			"type": "arrow",
			"version": 332,
			"versionNonce": 928296811,
			"isDeleted": false,
			"id": "5liynzCjkD7sv5v6KTBCE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -161.36751168420022,
			"y": 689.330359016085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 1.4396876481466734,
			"height": 55.75256628309228,
			"seed": 1583231499,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676284985,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "5NcHHLHHiA4sKfl4g52Ga",
				"gap": 8,
				"focus": 2.54031504431488e-16
			},
			"endBinding": {
				"elementId": "2J0ewkOFyfhyrJ1njN8pG",
				"gap": 1,
				"focus": 0.05534798175920312
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1.4396876481466734,
					55.75256628309228
				]
			]
		},
		{
			"type": "arrow",
			"version": 424,
			"versionNonce": 1207100203,
			"isDeleted": false,
			"id": "FriooeXG-GrTREil29gE8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -201.16457594787403,
			"y": 857.1878627847916,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 82.56059543659296,
			"height": 74.11319935629331,
			"seed": 1210764133,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "UARlYq9J"
				}
			],
			"updated": 1693676284989,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2J0ewkOFyfhyrJ1njN8pG",
				"gap": 13.104661830678722,
				"focus": -0.21355847497348163
			},
			"endBinding": {
				"elementId": "pBsRWwjDsaWHq43yAlHWj",
				"gap": 12.4169921875,
				"focus": -0.5548956119888596
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-82.56059543659296,
					74.11319935629331
				]
			]
		},
		{
			"type": "text",
			"version": 47,
			"versionNonce": 243593323,
			"isDeleted": false,
			"id": "UARlYq9J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -268.3180883614016,
			"y": 881.3508223215982,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 555029317,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693331000524,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "FriooeXG-GrTREil29gE8",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 482,
			"versionNonce": 1060009387,
			"isDeleted": false,
			"id": "i9ksv6doAZq733308n94t",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -107.33241369720932,
			"y": 841.7956752369593,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 139.1034262597641,
			"height": 88.64601190412566,
			"seed": 1250702219,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "rIaH69NZ"
				}
			],
			"updated": 1693676284992,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2J0ewkOFyfhyrJ1njN8pG",
				"gap": 11.398295158960622,
				"focus": -0.010805543889000744
			},
			"endBinding": {
				"elementId": "ppPVkQPP-xV5bbsCrbzZN",
				"gap": 13.2763671875,
				"focus": 0.6187844788824849
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					139.1034262597641,
					88.64601190412566
				]
			]
		},
		{
			"type": "text",
			"version": 45,
			"versionNonce": 1252896779,
			"isDeleted": false,
			"id": "rIaH69NZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -58.88067615326477,
			"y": 873.618681189022,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 42.199951171875,
			"height": 25,
			"seed": 1881928843,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693331004921,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "i9ksv6doAZq733308n94t",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 911,
			"versionNonce": 2031788869,
			"isDeleted": false,
			"id": "ZNpUrIcgDEuI_8Xh3csIh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -360.1831822878912,
			"y": 1136.074499641085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 170,
			"height": 100,
			"seed": 280047723,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "IUQSbQnv"
				},
				{
					"id": "olgR6bg-75pjYssV4AHs1",
					"type": "arrow"
				},
				{
					"id": "Py2U5P4hJoCM6JQuLbJi7",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1108,
			"versionNonce": 1062554981,
			"isDeleted": false,
			"id": "IUQSbQnv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -354.26316886015684,
			"y": 1148.574499641085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.15997314453125,
			"height": 75,
			"seed": 1336273675,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "基于CAS替换任务\n状态为COMPLET\nING",
			"rawText": "基于CAS替换任务状态为COMPLETING",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ZNpUrIcgDEuI_8Xh3csIh",
			"originalText": "基于CAS替换任务状态为COMPLETING",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "diamond",
			"version": 1638,
			"versionNonce": 1819302085,
			"isDeleted": false,
			"id": "qNMia7U0HpVX4hYBTmzVH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -357.1831822878912,
			"y": 1302.596472297335,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 164,
			"height": 120,
			"seed": 1226500165,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "GdQllVQs"
				},
				{
					"id": "Py2U5P4hJoCM6JQuLbJi7",
					"type": "arrow"
				},
				{
					"id": "VgR3yZEnR5xPxjerSQG1K",
					"type": "arrow"
				},
				{
					"id": "W4z6N-R80g68r-o81MhO4",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1992,
			"versionNonce": 1518292549,
			"isDeleted": false,
			"id": "GdQllVQs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -305.1831822878912,
			"y": 1337.596472297335,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 60,
			"height": 50,
			"seed": 612341669,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "替换成\n功？",
			"rawText": "替换成功？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "qNMia7U0HpVX4hYBTmzVH",
			"originalText": "替换成功？",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 1030,
			"versionNonce": 409419173,
			"isDeleted": false,
			"id": "1YyNu-ZHlq0TGkDAu-Z2r",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -360.1831822878912,
			"y": 1530.720007453585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 170,
			"height": 100,
			"seed": 708702021,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "aZtLVDU1"
				},
				{
					"id": "VgR3yZEnR5xPxjerSQG1K",
					"type": "arrow"
				},
				{
					"id": "GX3pY-Ci88xssJonSr188",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1256,
			"versionNonce": 2091353029,
			"isDeleted": false,
			"id": "aZtLVDU1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -351.62317709990293,
			"y": 1555.720007453585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 152.87998962402344,
			"height": 50,
			"seed": 430554789,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "修改任务状态为N\nORMAL",
			"rawText": "修改任务状态为NORMAL",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "1YyNu-ZHlq0TGkDAu-Z2r",
			"originalText": "修改任务状态为NORMAL",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 884,
			"versionNonce": 49070885,
			"isDeleted": false,
			"id": "ZMmKgwz_35pXMzZRtLCdc",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -55.08218488828197,
			"y": 1132.231726203585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 170,
			"height": 100,
			"seed": 689915141,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "c914oc4R"
				},
				{
					"id": "NWhTeskKHwHSoO2TJZBmi",
					"type": "arrow"
				},
				{
					"id": "4bXYWXNA3apAWytfHOIgX",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1050,
			"versionNonce": 1548286277,
			"isDeleted": false,
			"id": "c914oc4R",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -49.1621714605476,
			"y": 1144.731726203585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.15997314453125,
			"height": 75,
			"seed": 1700420709,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "基于CAS替换任务\n状态为COMPLET\nING",
			"rawText": "基于CAS替换任务状态为COMPLETING",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ZMmKgwz_35pXMzZRtLCdc",
			"originalText": "基于CAS替换任务状态为COMPLETING",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "diamond",
			"version": 1586,
			"versionNonce": 700680357,
			"isDeleted": false,
			"id": "p2bLCCbDusQt0RJS9n2WU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -52.08218488828197,
			"y": 1301.615026984835,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 164,
			"height": 120,
			"seed": 552771851,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "T5r7R9oR"
				},
				{
					"id": "4bXYWXNA3apAWytfHOIgX",
					"type": "arrow"
				},
				{
					"id": "BS045BQ75Ni8ZRoms111z",
					"type": "arrow"
				},
				{
					"id": "N3Ucs3RlpOa1XIxZWlZKK",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1943,
			"versionNonce": 978030117,
			"isDeleted": false,
			"id": "T5r7R9oR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -0.08218488828197223,
			"y": 1336.615026984835,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 60,
			"height": 50,
			"seed": 61884331,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "替换成\n功？",
			"rawText": "替换成功？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "p2bLCCbDusQt0RJS9n2WU",
			"originalText": "替换成功？",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 1102,
			"versionNonce": 1482719621,
			"isDeleted": false,
			"id": "m5kiGuH6B1GgqiyWl_1mS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -55.08218488828197,
			"y": 1531.681921516085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 170,
			"height": 100,
			"seed": 1416132229,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "yKgBnkGb"
				},
				{
					"id": "BS045BQ75Ni8ZRoms111z",
					"type": "arrow"
				},
				{
					"id": "nWbii04ElOMNkQ1nF0zeO",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1330,
			"versionNonce": 837079973,
			"isDeleted": false,
			"id": "yKgBnkGb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -46.8221827520515,
			"y": 1556.681921516085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 153.47999572753906,
			"height": 50,
			"seed": 58809829,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "修改任务状态为E\nXCEPTIONAL",
			"rawText": "修改任务状态为EXCEPTIONAL",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "m5kiGuH6B1GgqiyWl_1mS",
			"originalText": "修改任务状态为EXCEPTIONAL",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "arrow",
			"version": 273,
			"versionNonce": 1691142187,
			"isDeleted": false,
			"id": "olgR6bg-75pjYssV4AHs1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -275.1831822878912,
			"y": 1051.718054328585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0,
			"height": 76.3564453125,
			"seed": 725779595,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676284994,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "pBsRWwjDsaWHq43yAlHWj",
				"gap": 8,
				"focus": 0
			},
			"endBinding": {
				"elementId": "ZNpUrIcgDEuI_8Xh3csIh",
				"gap": 8,
				"focus": 0
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					76.3564453125
				]
			]
		},
		{
			"type": "arrow",
			"version": 518,
			"versionNonce": 292412075,
			"isDeleted": false,
			"id": "Py2U5P4hJoCM6JQuLbJi7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -274.4622576095695,
			"y": 1244.074499641085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0.6472832135367526,
			"height": 52.075379736661034,
			"seed": 1992111173,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676284997,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ZNpUrIcgDEuI_8Xh3csIh",
				"gap": 8,
				"focus": 2.618047143329404e-16
			},
			"endBinding": {
				"elementId": "qNMia7U0HpVX4hYBTmzVH",
				"gap": 6.59018612075856,
				"focus": 0.026757579132445444
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0.6472832135367526,
					52.075379736661034
				]
			]
		},
		{
			"type": "arrow",
			"version": 534,
			"versionNonce": 955830219,
			"isDeleted": false,
			"id": "VgR3yZEnR5xPxjerSQG1K",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -271.14296607405595,
			"y": 1430.4386572358135,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 2.4809239925258453,
			"height": 92.28135021777166,
			"seed": 1868789643,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285000,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qNMia7U0HpVX4hYBTmzVH",
				"gap": 8.821746520039824,
				"focus": -0.07151352642002612
			},
			"endBinding": {
				"elementId": "1YyNu-ZHlq0TGkDAu-Z2r",
				"gap": 7.999999999999773,
				"focus": -2.1076205688225907e-16
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-2.4809239925258453,
					92.28135021777166
				]
			]
		},
		{
			"type": "arrow",
			"version": 296,
			"versionNonce": 1141445195,
			"isDeleted": false,
			"id": "NWhTeskKHwHSoO2TJZBmi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 29.917815111718028,
			"y": 1051.718054328585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0,
			"height": 72.513671875,
			"seed": 805702827,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285002,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ppPVkQPP-xV5bbsCrbzZN",
				"gap": 8,
				"focus": 0
			},
			"endBinding": {
				"elementId": "ZMmKgwz_35pXMzZRtLCdc",
				"gap": 8,
				"focus": 0
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					72.513671875
				]
			]
		},
		{
			"type": "arrow",
			"version": 272,
			"versionNonce": 1436816587,
			"isDeleted": false,
			"id": "4bXYWXNA3apAWytfHOIgX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 29.881171007638756,
			"y": 1240.231726203585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0.03472661780051922,
			"height": 54.965017784956444,
			"seed": 2036386853,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285005,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ZMmKgwz_35pXMzZRtLCdc",
				"gap": 8,
				"focus": 1.7929161496914226e-16
			},
			"endBinding": {
				"elementId": "p2bLCCbDusQt0RJS9n2WU",
				"gap": 6.4186798019883256,
				"focus": -0.001382115295187062
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-0.03472661780051922,
					54.965017784956444
				]
			]
		},
		{
			"type": "arrow",
			"version": 570,
			"versionNonce": 90617323,
			"isDeleted": false,
			"id": "BS045BQ75Ni8ZRoms111z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 34.57338564284331,
			"y": 1434.9620774163072,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0.587041112249409,
			"height": 87.36109399974112,
			"seed": 1560297547,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285007,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "p2bLCCbDusQt0RJS9n2WU",
				"gap": 14.1357027483796,
				"focus": -0.05076463061932542
			},
			"endBinding": {
				"elementId": "m5kiGuH6B1GgqiyWl_1mS",
				"gap": 9.35875010003656,
				"focus": 0.06610909957585884
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0.587041112249409,
					87.36109399974112
				]
			]
		},
		{
			"type": "rectangle",
			"version": 1104,
			"versionNonce": 1829457733,
			"isDeleted": false,
			"id": "KF7Lws-CzvUSwJ5vlfVc_",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -191.8672459210727,
			"y": 1734.484655891085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 170,
			"height": 100,
			"seed": 1670288587,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "sPy1gUpM"
				},
				{
					"id": "GX3pY-Ci88xssJonSr188",
					"type": "arrow"
				},
				{
					"id": "nWbii04ElOMNkQ1nF0zeO",
					"type": "arrow"
				},
				{
					"id": "8u4L6Ug5AQl1-hIyYzNfW",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1409,
			"versionNonce": 2091582661,
			"isDeleted": false,
			"id": "sPy1gUpM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -176.8672459210727,
			"y": 1759.484655891085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140,
			"height": 50,
			"seed": 1041744747,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "开始唤醒等待结\n果的线程",
			"rawText": "开始唤醒等待结果的线程",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "KF7Lws-CzvUSwJ5vlfVc_",
			"originalText": "开始唤醒等待结果的线程",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 1156,
			"versionNonce": 416818213,
			"isDeleted": false,
			"id": "nasKDP_dJbTk0qfaY_svE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -191.8672459210727,
			"y": 1910.383093391085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 170,
			"height": 100,
			"seed": 1398329867,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "oho7mx0g"
				},
				{
					"id": "8u4L6Ug5AQl1-hIyYzNfW",
					"type": "arrow"
				},
				{
					"id": "Pf_86FZrZ0Wdc51lFdr5z",
					"type": "arrow"
				},
				{
					"id": "6bAT3i5on7VehP_KRbCtx",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1535,
			"versionNonce": 1383492005,
			"isDeleted": false,
			"id": "oho7mx0g",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -176.8672459210727,
			"y": 1922.883093391085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140,
			"height": 75,
			"seed": 1277712555,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "获取当前等待栈\n中的头节点直至\n为空",
			"rawText": "获取当前等待栈中的头节点直至为空",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "nasKDP_dJbTk0qfaY_svE",
			"originalText": "获取当前等待栈中的头节点直至为空",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1385,
			"versionNonce": 1839459589,
			"isDeleted": false,
			"id": "S0mtAgNgv2gj84bMGMF34",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -191.8672459210727,
			"y": 2265.9895387035854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 170,
			"height": 100,
			"seed": 123785509,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "zd3VOJAR"
				},
				{
					"id": "sgW2_NYAg1ho1u_1SrM0a",
					"type": "arrow"
				},
				{
					"id": "XmcDbR_5odqvSh1KTpBLy",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1832,
			"versionNonce": 369487653,
			"isDeleted": false,
			"id": "zd3VOJAR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -176.8672459210727,
			"y": 2278.4895387035854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140,
			"height": 75,
			"seed": 2010473605,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "若该节点线程不\n为空，唤醒该线\n程",
			"rawText": "若该节点线程不为空，唤醒该线程",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "S0mtAgNgv2gj84bMGMF34",
			"originalText": "若该节点线程不为空，唤醒该线程",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1444,
			"versionNonce": 1881588357,
			"isDeleted": false,
			"id": "SpV50twEzkKkPYo_ODNy2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -191.8672459210727,
			"y": 2431.5442262035854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 170,
			"height": 100,
			"seed": 176654219,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "adq0cLAN"
				},
				{
					"id": "sgW2_NYAg1ho1u_1SrM0a",
					"type": "arrow"
				},
				{
					"id": "ALlqjU5L7Zw6UoxrDp4hx",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1879,
			"versionNonce": 1088861349,
			"isDeleted": false,
			"id": "adq0cLAN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -182.62723279851411,
			"y": 2456.5442262035854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 151.5199737548828,
			"height": 50,
			"seed": 1726242347,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "获取该节点的nex\nt节点",
			"rawText": "获取该节点的next节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "SpV50twEzkKkPYo_ODNy2",
			"originalText": "获取该节点的next节点",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 1485,
			"versionNonce": 1574098949,
			"isDeleted": false,
			"id": "uR8_bHUA5-psCFRtH350y",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -191.8672459210727,
			"y": 2585.747676724419,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 170,
			"height": 100,
			"seed": 173960171,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "FArjZvq6"
				},
				{
					"id": "ALlqjU5L7Zw6UoxrDp4hx",
					"type": "arrow"
				},
				{
					"id": "6bAT3i5on7VehP_KRbCtx",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2038,
			"versionNonce": 7911973,
			"isDeleted": false,
			"id": "FArjZvq6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -185.94723249333833,
			"y": 2598.247676724419,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.15997314453125,
			"height": 75,
			"seed": 656389771,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "利用CAS操作将其\n将其替换为新的\n栈顶节点",
			"rawText": "利用CAS操作将其将其替换为新的栈顶节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "uR8_bHUA5-psCFRtH350y",
			"originalText": "利用CAS操作将其将其替换为新的栈顶节点",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1564,
			"versionNonce": 940060037,
			"isDeleted": false,
			"id": "bd7lEcYF7HQVmmhbHB9-x",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -191.8672459210727,
			"y": 2731.5461793285854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#96f2d7",
			"width": 170,
			"height": 100,
			"seed": 1073795691,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "d77UtmCp"
				},
				{
					"id": "WigdQYampp78KNqIgRnHx",
					"type": "arrow"
				},
				{
					"id": "_OqOTBbw4TSULsJrIW23O",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2144,
			"versionNonce": 1377566629,
			"isDeleted": false,
			"id": "d77UtmCp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -146.8672459210727,
			"y": 2769.0461793285854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 80,
			"height": 25,
			"seed": 2024885515,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "唤醒结束",
			"rawText": "唤醒结束",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "bd7lEcYF7HQVmmhbHB9-x",
			"originalText": "唤醒结束",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 267,
			"versionNonce": 834857067,
			"isDeleted": false,
			"id": "GX3pY-Ci88xssJonSr188",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -227.2733785798656,
			"y": 1638.720007453585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 72.49632895076729,
			"height": 87.7646484375,
			"seed": 516738827,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285010,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1YyNu-ZHlq0TGkDAu-Z2r",
				"gap": 8,
				"focus": 1.6409745830359918e-16
			},
			"endBinding": {
				"elementId": "KF7Lws-CzvUSwJ5vlfVc_",
				"gap": 8,
				"focus": 1.6409745830359918e-16
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					72.49632895076729,
					87.7646484375
				]
			]
		},
		{
			"type": "arrow",
			"version": 273,
			"versionNonce": 862849451,
			"isDeleted": false,
			"id": "nWbii04ElOMNkQ1nF0zeO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -9.201645305576214,
			"y": 1639.681921516085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 58.54614019820222,
			"height": 86.80273437500023,
			"seed": 859181861,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285011,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "m5kiGuH6B1GgqiyWl_1mS",
				"gap": 8,
				"focus": -8.825299323660783e-17
			},
			"endBinding": {
				"elementId": "KF7Lws-CzvUSwJ5vlfVc_",
				"gap": 7.999999999999773,
				"focus": 2.2063248309151956e-15
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-58.54614019820222,
					86.80273437500023
				]
			]
		},
		{
			"type": "arrow",
			"version": 251,
			"versionNonce": 44194859,
			"isDeleted": false,
			"id": "8u4L6Ug5AQl1-hIyYzNfW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -106.86724592107271,
			"y": 1842.484655891085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0,
			"height": 59.8984375,
			"seed": 1428916587,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285013,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "KF7Lws-CzvUSwJ5vlfVc_",
				"gap": 8,
				"focus": 0
			},
			"endBinding": {
				"elementId": "nasKDP_dJbTk0qfaY_svE",
				"gap": 8,
				"focus": 0
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					59.8984375
				]
			]
		},
		{
			"type": "arrow",
			"version": 794,
			"versionNonce": 1806164939,
			"isDeleted": false,
			"id": "sgW2_NYAg1ho1u_1SrM0a",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -106.86724592107271,
			"y": 2373.9895387035854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0,
			"height": 49.5546875,
			"seed": 819312619,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285030,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "S0mtAgNgv2gj84bMGMF34",
				"gap": 8,
				"focus": 0
			},
			"endBinding": {
				"elementId": "SpV50twEzkKkPYo_ODNy2",
				"gap": 8,
				"focus": 0
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					49.5546875
				]
			]
		},
		{
			"type": "arrow",
			"version": 820,
			"versionNonce": 747513419,
			"isDeleted": false,
			"id": "ALlqjU5L7Zw6UoxrDp4hx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -106.86724592107271,
			"y": 2539.5442262035854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0,
			"height": 38.203450520833485,
			"seed": 261998699,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285032,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "SpV50twEzkKkPYo_ODNy2",
				"gap": 8,
				"focus": 0
			},
			"endBinding": {
				"elementId": "uR8_bHUA5-psCFRtH350y",
				"gap": 8,
				"focus": 0
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					38.203450520833485
				]
			]
		},
		{
			"type": "diamond",
			"version": 1648,
			"versionNonce": 831733733,
			"isDeleted": false,
			"id": "OZSp5buXUa-cZ0uwvWW0Z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -188.8672459210727,
			"y": 2050.583288703585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 164,
			"height": 120,
			"seed": 1966953957,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "mVuUfApF"
				},
				{
					"id": "Pf_86FZrZ0Wdc51lFdr5z",
					"type": "arrow"
				},
				{
					"id": "XmcDbR_5odqvSh1KTpBLy",
					"type": "arrow"
				},
				{
					"id": "WigdQYampp78KNqIgRnHx",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2023,
			"versionNonce": 1374004581,
			"isDeleted": false,
			"id": "mVuUfApF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -136.8672459210727,
			"y": 2085.583288703585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 60,
			"height": 50,
			"seed": 692277573,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "节点为\n空？",
			"rawText": "节点为空？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "OZSp5buXUa-cZ0uwvWW0Z",
			"originalText": "节点为空？",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "arrow",
			"version": 257,
			"versionNonce": 840806923,
			"isDeleted": false,
			"id": "Pf_86FZrZ0Wdc51lFdr5z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -106.61770289418179,
			"y": 2018.3830933910854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0.13456783408990702,
			"height": 31.276908333019946,
			"seed": 631006155,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285038,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "nasKDP_dJbTk0qfaY_svE",
				"gap": 8,
				"focus": 6.09166362708838e-16
			},
			"endBinding": {
				"elementId": "OZSp5buXUa-cZ0uwvWW0Z",
				"gap": 1,
				"focus": 0.007880868666504495
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0.13456783408990702,
					31.276908333019946
				]
			]
		},
		{
			"type": "arrow",
			"version": 460,
			"versionNonce": 1973087051,
			"isDeleted": false,
			"id": "XmcDbR_5odqvSh1KTpBLy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -108.73862696355914,
			"y": 2180.274034751039,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 1.2656738189408827,
			"height": 74.58399353587947,
			"seed": 100482437,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "jBzy7z86"
				}
			],
			"updated": 1693676285038,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "OZSp5buXUa-cZ0uwvWW0Z",
				"gap": 9.869783481030368,
				"focus": 0.03724411631021271
			},
			"endBinding": {
				"elementId": "S0mtAgNgv2gj84bMGMF34",
				"gap": 11.13151041666697,
				"focus": 0.005028404666061201
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1.2656738189408827,
					74.58399353587947
				]
			]
		},
		{
			"type": "text",
			"version": 45,
			"versionNonce": 1773798277,
			"isDeleted": false,
			"id": "jBzy7z86",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -133.1258090688874,
			"y": 2206.994421516085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 839225451,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "XmcDbR_5odqvSh1KTpBLy",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 330,
			"versionNonce": 875111659,
			"isDeleted": false,
			"id": "6bAT3i5on7VehP_KRbCtx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -13.085019358572595,
			"y": 2645.751783485841,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 88.05246727523924,
			"height": 686.7882498117388,
			"seed": 706414501,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "NTvg7Q5p"
				}
			],
			"updated": 1693676285032,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "uR8_bHUA5-psCFRtH350y",
				"gap": 8.782226562500128,
				"focus": 0.9874241670201114
			},
			"endBinding": {
				"elementId": "nasKDP_dJbTk0qfaY_svE",
				"gap": 12.577473958333357,
				"focus": -1.0036450910875783
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					88.05246727523924,
					-351.873898428089
				],
				[
					3.7952473958332575,
					-686.7882498117388
				]
			]
		},
		{
			"type": "text",
			"version": 43,
			"versionNonce": 265856581,
			"isDeleted": false,
			"id": "NTvg7Q5p",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 54.96744791666663,
			"y": 2281.377885057752,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 40,
			"height": 25,
			"seed": 1036313483,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "自旋",
			"rawText": "自旋",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "6bAT3i5on7VehP_KRbCtx",
			"originalText": "自旋",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 378,
			"versionNonce": 693455339,
			"isDeleted": false,
			"id": "WigdQYampp78KNqIgRnHx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -188.41883779255767,
			"y": 2127.1882992879664,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 94.2699642907757,
			"height": 674.966277059932,
			"seed": 1235776139,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Jv0IbGhS"
				}
			],
			"updated": 1693676285038,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "OZSp5buXUa-cZ0uwvWW0Z",
				"gap": 13.135957618023816,
				"focus": 0.9361258466710785
			},
			"endBinding": {
				"elementId": "bd7lEcYF7HQVmmhbHB9-x",
				"gap": 14.463216145833371,
				"focus": -1.0835319165043789
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-94.2699642907757,
					326.84583576978594
				],
				[
					-17.91162427434847,
					674.966277059932
				]
			]
		},
		{
			"type": "text",
			"version": 39,
			"versionNonce": 1732090117,
			"isDeleted": false,
			"id": "Jv0IbGhS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -294.26273600260424,
			"y": 2440.679968391086,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 42.199951171875,
			"height": 25,
			"seed": 1839612395,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "WigdQYampp78KNqIgRnHx",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "ellipse",
			"version": 1090,
			"versionNonce": 1955475557,
			"isDeleted": false,
			"id": "gE95kXf-n_snmYUmC1Pp-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -197.8672459210727,
			"y": 3606.4100157820953,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 182,
			"height": 140,
			"seed": 779162565,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AElEUsEH"
				},
				{
					"id": "Z5T3T2vZZ_tgdou1oEGWO",
					"type": "arrow"
				},
				{
					"id": "EQSTwKuYQvqX-8oWH3a1J",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 993,
			"versionNonce": 1770679941,
			"isDeleted": false,
			"id": "AElEUsEH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -126.71396300904854,
			"y": 3663.912541099037,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 40,
			"height": 25,
			"seed": 1694782245,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "结束",
			"rawText": "结束",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "gE95kXf-n_snmYUmC1Pp-",
			"originalText": "结束",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "diamond",
			"version": 1830,
			"versionNonce": 295982565,
			"isDeleted": false,
			"id": "WoccgP5YWM7TW75v4zVQ1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -237.3672459210727,
			"y": 2946.65514719674,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 261,
			"height": 253,
			"seed": 137626149,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Vvhjt22S"
				},
				{
					"id": "qoSrMHFZwhnT22ia7jHDd",
					"type": "arrow"
				},
				{
					"id": "EQSTwKuYQvqX-8oWH3a1J",
					"type": "arrow"
				},
				{
					"id": "_OqOTBbw4TSULsJrIW23O",
					"type": "arrow"
				},
				{
					"id": "N3Ucs3RlpOa1XIxZWlZKK",
					"type": "arrow"
				},
				{
					"id": "W4z6N-R80g68r-o81MhO4",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2321,
			"versionNonce": 1257629221,
			"isDeleted": false,
			"id": "Vvhjt22S",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -166.6172459210727,
			"y": 3035.90514719674,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 120,
			"height": 75,
			"seed": 958035333,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "当前任务状态\n为INTERRU\nPTING？",
			"rawText": "当前任务状态为INTERRUPTING？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "WoccgP5YWM7TW75v4zVQ1",
			"originalText": "当前任务状态为INTERRUPTING？",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1764,
			"versionNonce": 301980037,
			"isDeleted": false,
			"id": "9KIiTx14Y40tHYQoe01Bw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -192.84099016606024,
			"y": 3386.9220192588527,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 170,
			"height": 100,
			"seed": 1244655557,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Ns8bSZRe"
				},
				{
					"id": "qoSrMHFZwhnT22ia7jHDd",
					"type": "arrow"
				},
				{
					"id": "Z5T3T2vZZ_tgdou1oEGWO",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2388,
			"versionNonce": 1319316389,
			"isDeleted": false,
			"id": "Ns8bSZRe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -177.84099016606024,
			"y": 3411.9220192588527,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140,
			"height": 50,
			"seed": 1552396069,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "让出当前线程执\n行权",
			"rawText": "让出当前线程执行权",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "9KIiTx14Y40tHYQoe01Bw",
			"originalText": "让出当前线程执行权",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "arrow",
			"version": 643,
			"versionNonce": 1736712555,
			"isDeleted": false,
			"id": "qoSrMHFZwhnT22ia7jHDd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -110.1920198905776,
			"y": 3207.3490403323417,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 3.4086941628072225,
			"height": 170.339743825582,
			"seed": 330258379,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "ji8Uhr8l"
				}
			],
			"updated": 1693676285046,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "WoccgP5YWM7TW75v4zVQ1",
				"gap": 8.381534079770674,
				"focus": 0.046054774656471176
			},
			"endBinding": {
				"elementId": "9KIiTx14Y40tHYQoe01Bw",
				"gap": 9.23323510092905,
				"focus": 0.0260811052329668
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					3.4086941628072225,
					170.339743825582
				]
			]
		},
		{
			"type": "text",
			"version": 110,
			"versionNonce": 1215186533,
			"isDeleted": false,
			"id": "ji8Uhr8l",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -124.87284562531136,
			"y": 3326.551386647112,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 42.199951171875,
			"height": 25,
			"seed": 1448776581,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "qoSrMHFZwhnT22ia7jHDd",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 506,
			"versionNonce": 1391788715,
			"isDeleted": false,
			"id": "Z5T3T2vZZ_tgdou1oEGWO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -106.50817474068523,
			"y": 3494.922019258853,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 2.417407387915077,
			"height": 105.19808356781414,
			"seed": 1701350123,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285046,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "9KIiTx14Y40tHYQoe01Bw",
				"gap": 8,
				"focus": -7.213077627821023e-17
			},
			"endBinding": {
				"elementId": "gE95kXf-n_snmYUmC1Pp-",
				"gap": 6.3209784339432815,
				"focus": 0.04976792249906975
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2.417407387915077,
					105.19808356781414
				]
			]
		},
		{
			"type": "arrow",
			"version": 520,
			"versionNonce": 6135211,
			"isDeleted": false,
			"id": "EQSTwKuYQvqX-8oWH3a1J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -10.8765435759436,
			"y": 3136.686909018741,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 99.97358023520344,
			"height": 542.0891140471895,
			"seed": 921732363,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "a1dXJYmM"
				}
			],
			"updated": 1693676285043,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "WoccgP5YWM7TW75v4zVQ1",
				"gap": 21.598361414420893,
				"focus": -0.571551993201427
			},
			"endBinding": {
				"elementId": "gE95kXf-n_snmYUmC1Pp-",
				"gap": 14.222946102781151,
				"focus": 1.1209346826151467
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					99.97358023520344,
					296.75503718897016
				],
				[
					9.190716097001484,
					542.0891140471895
				]
			]
		},
		{
			"type": "text",
			"version": 110,
			"versionNonce": 773473413,
			"isDeleted": false,
			"id": "a1dXJYmM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 64.0370619888497,
			"y": 3420.941946207711,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 674813195,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "EQSTwKuYQvqX-8oWH3a1J",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 485,
			"versionNonce": 1145859819,
			"isDeleted": false,
			"id": "_OqOTBbw4TSULsJrIW23O",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -107.35349365269394,
			"y": 2839.5461793285854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 0.8705074215059057,
			"height": 103.83478865598317,
			"seed": 1083899525,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285043,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "bd7lEcYF7HQVmmhbHB9-x",
				"gap": 8,
				"focus": -4.8228894231297415e-18
			},
			"endBinding": {
				"elementId": "WoccgP5YWM7TW75v4zVQ1",
				"gap": 3.544154914637943,
				"focus": -0.018733544609102846
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-0.8705074215059057,
					103.83478865598317
				]
			]
		},
		{
			"type": "arrow",
			"version": 563,
			"versionNonce": 441801099,
			"isDeleted": false,
			"id": "N3Ucs3RlpOa1XIxZWlZKK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 116.56914429872366,
			"y": 1361.615026984835,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 255.94883458142743,
			"height": 1701.0259588533954,
			"seed": 1735809227,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "qPv7TiEr"
				}
			],
			"updated": 1693676285043,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "p2bLCCbDusQt0RJS9n2WU",
				"gap": 2.7466563720978456,
				"focus": -1.0567235266708004
			},
			"endBinding": {
				"elementId": "WoccgP5YWM7TW75v4zVQ1",
				"gap": 11.844943469960583,
				"focus": 1.0217182939859
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					169.1840023757124,
					894.6643204590273
				],
				[
					-86.76483220571502,
					1701.0259588533954
				]
			]
		},
		{
			"type": "text",
			"version": 34,
			"versionNonce": 313620133,
			"isDeleted": false,
			"id": "qPv7TiEr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 260.6931720040259,
			"y": 2243.779347443862,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 2086046827,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "N3Ucs3RlpOa1XIxZWlZKK",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 946,
			"versionNonce": 556265515,
			"isDeleted": false,
			"id": "W4z6N-R80g68r-o81MhO4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -369.7437731991382,
			"y": 1362.596472297335,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 319.0163140685627,
			"height": 1689.172861361305,
			"seed": 197566629,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "gxoBKnld"
				}
			],
			"updated": 1693676285044,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qNMia7U0HpVX4hYBTmzVH",
				"gap": 7.417154468462918,
				"focus": 1.1531779379420366
			},
			"endBinding": {
				"elementId": "WoccgP5YWM7TW75v4zVQ1",
				"gap": 11.729891943869717,
				"focus": -0.8948181806419973
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-181.4306250712527,
					888.1430650671196
				],
				[
					137.58568899730997,
					1689.172861361305
				]
			]
		},
		{
			"type": "text",
			"version": 67,
			"versionNonce": 1336267109,
			"isDeleted": false,
			"id": "gxoBKnld",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -576.2343729408011,
			"y": 2238.239537364454,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 1816148171,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330997578,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "W4z6N-R80g68r-o81MhO4",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "ellipse",
			"version": 1159,
			"versionNonce": 672160965,
			"isDeleted": false,
			"id": "r4ct2buhKKsXXFHoClvip",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -552.6248763536391,
			"y": 622.500353645,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 182,
			"height": 140,
			"seed": 785414373,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "w62p7tP1"
				},
				{
					"id": "xiqDN8tr8xDqGdVK9nthw",
					"type": "arrow"
				},
				{
					"id": "t-Q1Gt6WwGi1-NB1BQrfV",
					"type": "arrow"
				}
			],
			"updated": 1693330997578,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1063,
			"versionNonce": 1216648197,
			"isDeleted": false,
			"id": "w62p7tP1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -481.4715934416149,
			"y": 680.0028789619416,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 40,
			"height": 25,
			"seed": 712643653,
			"groupIds": [
				"VaZ8qDj_bZCgDJhPBF5mA"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693411314039,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "结束",
			"rawText": "结束",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "r4ct2buhKKsXXFHoClvip",
			"originalText": "结束",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 435,
			"versionNonce": 530268907,
			"isDeleted": false,
			"id": "3EFNslUPy8xkahgzoXI6C",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1255.1404772762899,
			"y": -157.93938180063014,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 170,
			"height": 100,
			"seed": 143660197,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "p2O955cT"
				},
				{
					"id": "HyKghbtfjl-9NCrGnMiiL",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 383,
			"versionNonce": 752809285,
			"isDeleted": false,
			"id": "p2O955cT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1296.4805041317586,
			"y": -120.43938180063014,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 87.3199462890625,
			"height": 25,
			"seed": 693123077,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "get()调用",
			"rawText": "get()调用",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "3EFNslUPy8xkahgzoXI6C",
			"originalText": "get()调用",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "diamond",
			"version": 1451,
			"versionNonce": 1286188427,
			"isDeleted": false,
			"id": "_zeglng8hhlkfc91gWGTx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1231.4655980437055,
			"y": -2.0343794819526124,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 222,
			"height": 220,
			"seed": 722859275,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AusEUnF9"
				},
				{
					"id": "HyKghbtfjl-9NCrGnMiiL",
					"type": "arrow"
				},
				{
					"id": "NVDpkPkGGvlHrlO6gRUCz",
					"type": "arrow"
				},
				{
					"id": "du5CYFh9M0HyS91ANM-Kb",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1797,
			"versionNonce": 1849296037,
			"isDeleted": false,
			"id": "AusEUnF9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1292.4655980437055,
			"y": 57.96562051804739,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 100,
			"height": 100,
			"seed": 373889963,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "任务执行状\n态为New或\nCOMPLET\nING？",
			"rawText": "任务执行状态为New或COMPLETING？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "_zeglng8hhlkfc91gWGTx",
			"originalText": "任务执行状态为New或COMPLETING？",
			"lineHeight": 1.25,
			"baseline": 93
		},
		{
			"type": "ellipse",
			"version": 2077,
			"versionNonce": 768721963,
			"isDeleted": false,
			"id": "HPoLvy7-v6RD4ZpgP9BHf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 765.9585501651304,
			"y": 1044.2775367538572,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 182,
			"height": 140,
			"seed": 1976284549,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "UtomQQ6i"
				},
				{
					"id": "eG2-OWdf0NS1ai2QswucL",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2002,
			"versionNonce": 607922181,
			"isDeleted": false,
			"id": "UtomQQ6i",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 817.1118330771546,
			"y": 1101.780062070799,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 80,
			"height": 25,
			"seed": 874822885,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "返回结果",
			"rawText": "返回结果",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "HPoLvy7-v6RD4ZpgP9BHf",
			"originalText": "返回结果",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 520,
			"versionNonce": 1795032779,
			"isDeleted": false,
			"id": "QqxpVXU6jBHMhdwIyN_ds",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1257.7696520590757,
			"y": 294.31467937347145,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 170,
			"height": 100,
			"seed": 460016741,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "kgXqSavI"
				},
				{
					"id": "NVDpkPkGGvlHrlO6gRUCz",
					"type": "arrow"
				},
				{
					"id": "Sy1LkjGc2u3LKLtaLzWq5",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 513,
			"versionNonce": 1357045605,
			"isDeleted": false,
			"id": "kgXqSavI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1282.7696520590757,
			"y": 319.31467937347145,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 120,
			"height": 50,
			"seed": 1936619461,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "开始自旋等待\n任务完成",
			"rawText": "开始自旋等待\n任务完成",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "QqxpVXU6jBHMhdwIyN_ds",
			"originalText": "开始自旋等待\n任务完成",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "diamond",
			"version": 1631,
			"versionNonce": 1254684011,
			"isDeleted": false,
			"id": "lkyh9LAPYWEAPEXVlxwuk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1239.0630126147137,
			"y": 460.595669485597,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 201,
			"height": 171,
			"seed": 1146766827,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "K350siRl"
				},
				{
					"id": "Sy1LkjGc2u3LKLtaLzWq5",
					"type": "arrow"
				},
				{
					"id": "fw-W4KCVrqY1ic3-YZHZU",
					"type": "arrow"
				},
				{
					"id": "32TOo9snRU9wGK-EjaRZk",
					"type": "arrow"
				},
				{
					"id": "6VShRYN64sa7XnbIvn5Pf",
					"type": "arrow"
				},
				{
					"id": "pYC8J0Ppbf6fQfYIQFi9-",
					"type": "arrow"
				},
				{
					"id": "Zwit6HcoY4nMFXE21fQ4N",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2040,
			"versionNonce": 1518054085,
			"isDeleted": false,
			"id": "K350siRl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1299.8130126147137,
			"y": 521.345669485597,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 80,
			"height": 50,
			"seed": 680546443,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "判断任务\n当前状态",
			"rawText": "判断任务\n当前状态",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "lkyh9LAPYWEAPEXVlxwuk",
			"originalText": "判断任务\n当前状态",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 828,
			"versionNonce": 1551699979,
			"isDeleted": false,
			"id": "mFxwk24xTJuxbLzKcMrU9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 774.6101630327978,
			"y": 783.0829190525482,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 170,
			"height": 100,
			"seed": 1384976011,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Tfawvd1v"
				},
				{
					"id": "fw-W4KCVrqY1ic3-YZHZU",
					"type": "arrow"
				},
				{
					"id": "eG2-OWdf0NS1ai2QswucL",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 909,
			"versionNonce": 27527717,
			"isDeleted": false,
			"id": "Tfawvd1v",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 784.0001776812353,
			"y": 795.5829190525482,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 151.219970703125,
			"height": 75,
			"seed": 1117848875,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "若等待节点线程\n不为null，将其置\n为null",
			"rawText": "若等待节点线程不为null，将其置为null",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "mFxwk24xTJuxbLzKcMrU9",
			"originalText": "若等待节点线程不为null，将其置为null",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1317,
			"versionNonce": 1645532843,
			"isDeleted": false,
			"id": "my3x8EDv5uIfv10cYys0w",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1065.8000750656236,
			"y": 784.6419369473773,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 170,
			"height": 100,
			"seed": 1225590309,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AM5nj69Y"
				},
				{
					"id": "32TOo9snRU9wGK-EjaRZk",
					"type": "arrow"
				},
				{
					"id": "Zwit6HcoY4nMFXE21fQ4N",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1475,
			"versionNonce": 390692229,
			"isDeleted": false,
			"id": "AM5nj69Y",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1071.720088493358,
			"y": 797.1419369473773,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.15997314453125,
			"height": 75,
			"seed": 3340677,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "构建等待节点并\n基于CAS加入等待\n队列中",
			"rawText": "构建等待节点并基于CAS加入等待队列中",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "my3x8EDv5uIfv10cYys0w",
			"originalText": "构建等待节点并基于CAS加入等待队列中",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 905,
			"versionNonce": 2050672971,
			"isDeleted": false,
			"id": "1OdsebFJjJV_VKSO0aQAk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1578.1741607313295,
			"y": 793.6597855547227,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 170,
			"height": 100,
			"seed": 1054217637,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "fMmcpUdM"
				},
				{
					"id": "6VShRYN64sa7XnbIvn5Pf",
					"type": "arrow"
				},
				{
					"id": "mN3EqTUfaXp2NNMo0DCjh",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1095,
			"versionNonce": 1923712229,
			"isDeleted": false,
			"id": "fMmcpUdM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1603.1741607313295,
			"y": 831.1597855547227,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 120,
			"height": 25,
			"seed": 154016005,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "移除等待节点",
			"rawText": "移除等待节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "1OdsebFJjJV_VKSO0aQAk",
			"originalText": "移除等待节点",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 496,
			"versionNonce": 532529419,
			"isDeleted": false,
			"id": "HyKghbtfjl-9NCrGnMiiL",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1338.778851917973,
			"y": -54.59832730919317,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 0.9358079735338833,
			"height": 51.419924894975594,
			"seed": 1051798411,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285053,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "3EFNslUPy8xkahgzoXI6C",
				"gap": 3.341054491436978,
				"focus": 0.02714931335780351
			},
			"endBinding": {
				"elementId": "_zeglng8hhlkfc91gWGTx",
				"gap": 2.7489838023526545,
				"focus": -0.0065602869491984805
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0.9358079735338833,
					51.419924894975594
				]
			]
		},
		{
			"type": "arrow",
			"version": 524,
			"versionNonce": 527681035,
			"isDeleted": false,
			"id": "NVDpkPkGGvlHrlO6gRUCz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1345.5712004507866,
			"y": 222.4243093816418,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 0.559302679918801,
			"height": 63.211744095791545,
			"seed": 472988011,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "hob9cdvw"
				}
			],
			"updated": 1693676285057,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "_zeglng8hhlkfc91gWGTx",
				"gap": 5.433661076402174,
				"focus": -0.01885461774714612
			},
			"endBinding": {
				"elementId": "QqxpVXU6jBHMhdwIyN_ds",
				"gap": 8.678625896038113,
				"focus": 0.045411226340221836
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0.559302679918801,
					63.211744095791545
				]
			]
		},
		{
			"type": "text",
			"version": 106,
			"versionNonce": 1749758603,
			"isDeleted": false,
			"id": "hob9cdvw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1324.751374005742,
			"y": 241.58644231716062,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffa94d",
			"width": 42.199951171875,
			"height": 25,
			"seed": 99678149,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "NVDpkPkGGvlHrlO6gRUCz",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 571,
			"versionNonce": 1463216267,
			"isDeleted": false,
			"id": "Sy1LkjGc2u3LKLtaLzWq5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1338.3036853844942,
			"y": 401.0573215782304,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 0.6867839765181998,
			"height": 54.42483726698623,
			"seed": 2000346053,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285060,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "QqxpVXU6jBHMhdwIyN_ds",
				"gap": 6.742642204758965,
				"focus": 0.04379181323730467
			},
			"endBinding": {
				"elementId": "lkyh9LAPYWEAPEXVlxwuk",
				"gap": 5.471319758364714,
				"focus": -0.030741870924128424
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-0.6867839765181998,
					54.42483726698623
				]
			]
		},
		{
			"type": "arrow",
			"version": 1218,
			"versionNonce": 1974300043,
			"isDeleted": false,
			"id": "fw-W4KCVrqY1ic3-YZHZU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1227.6182350077183,
			"y": 538.4571280545833,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 334.2380913323839,
			"height": 225.58269960268206,
			"seed": 2108405957,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "s8c4PjMX"
				}
			],
			"updated": 1693676285064,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "lkyh9LAPYWEAPEXVlxwuk",
				"gap": 13.759732906815131,
				"focus": 0.7207200732240681
			},
			"endBinding": {
				"elementId": "mFxwk24xTJuxbLzKcMrU9",
				"gap": 19.043091395282772,
				"focus": -0.33641199778711495
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-152.89139093260417,
					73.72867166639378
				],
				[
					-334.2380913323839,
					225.58269960268206
				]
			]
		},
		{
			"type": "text",
			"version": 251,
			"versionNonce": 473961221,
			"isDeleted": false,
			"id": "s8c4PjMX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1044.1376217678776,
			"y": 650.473459654659,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 200,
			"height": 25,
			"seed": 382394635,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "任务已正常或异常结束",
			"rawText": "任务已正常或异常结束",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "fw-W4KCVrqY1ic3-YZHZU",
			"originalText": "任务已正常或异常结束",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 1876,
			"versionNonce": 94425099,
			"isDeleted": false,
			"id": "32TOo9snRU9wGK-EjaRZk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1299.533346889803,
			"y": 610.0264274234777,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 166.51866067870287,
			"height": 155.75957626175273,
			"seed": 444852715,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "eb879XU4"
				}
			],
			"updated": 1693676285066,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "lkyh9LAPYWEAPEXVlxwuk",
				"gap": 9.509954476970208,
				"focus": -0.2817622485470041
			},
			"endBinding": {
				"elementId": "my3x8EDv5uIfv10cYys0w",
				"gap": 18.85593326214689,
				"focus": -0.6601305041818563
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-166.51866067870287,
					155.75957626175273
				]
			]
		},
		{
			"type": "text",
			"version": 165,
			"versionNonce": 389114469,
			"isDeleted": false,
			"id": "eb879XU4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1183.2322671206425,
			"y": 656.4538217082684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 200,
			"height": 25,
			"seed": 735831851,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "未完成，构建等待节点",
			"rawText": "未完成，构建等待节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "32TOo9snRU9wGK-EjaRZk",
			"originalText": "未完成，构建等待节点",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 827,
			"versionNonce": 1052547691,
			"isDeleted": false,
			"id": "c7-V7H-llPAvBYvbszZY1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1852.2855918166106,
			"y": 793.7148097157167,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffa94d",
			"width": 170,
			"height": 100,
			"seed": 502635563,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "SxceXMks"
				},
				{
					"id": "pYC8J0Ppbf6fQfYIQFi9-",
					"type": "arrow"
				}
			],
			"updated": 1693330996457,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1057,
			"versionNonce": 213964229,
			"isDeleted": false,
			"id": "SxceXMks",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1897.2855918166106,
			"y": 818.7148097157167,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 80,
			"height": 50,
			"seed": 1491525323,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "开始阻塞\n等待唤醒",
			"rawText": "开始阻塞\n等待唤醒",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "c7-V7H-llPAvBYvbszZY1",
			"originalText": "开始阻塞\n等待唤醒",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "arrow",
			"version": 926,
			"versionNonce": 1896163307,
			"isDeleted": false,
			"id": "6VShRYN64sa7XnbIvn5Pf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1400.0506371143176,
			"y": 612.9591465848134,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 240.37279021280392,
			"height": 171.03653894200636,
			"seed": 643800235,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "KYVg5yeB"
				}
			],
			"updated": 1693676285069,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "lkyh9LAPYWEAPEXVlxwuk",
				"gap": 25,
				"focus": 0.27863963802735764
			},
			"endBinding": {
				"elementId": "1OdsebFJjJV_VKSO0aQAk",
				"gap": 9.664100027902919,
				"focus": 0.3935128108957274
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					240.37279021280392,
					171.03653894200636
				]
			]
		},
		{
			"type": "text",
			"version": 148,
			"versionNonce": 671864101,
			"isDeleted": false,
			"id": "KYVg5yeB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1419.2036673782795,
			"y": 653.0469162925245,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 160,
			"height": 25,
			"seed": 345710309,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "已超时或已被中断",
			"rawText": "已超时或已被中断",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "6VShRYN64sa7XnbIvn5Pf",
			"originalText": "已超时或已被中断",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 672,
			"versionNonce": 1091963851,
			"isDeleted": false,
			"id": "pYC8J0Ppbf6fQfYIQFi9-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1454.5911220505288,
			"y": 559.1660031173094,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 472.81986260569374,
			"height": 222.2618882297918,
			"seed": 1231422597,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "26l9fjxB"
				}
			],
			"updated": 1693676285071,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "lkyh9LAPYWEAPEXVlxwuk",
				"gap": 19.54225127827631,
				"focus": -0.21769650001604482
			},
			"endBinding": {
				"elementId": "c7-V7H-llPAvBYvbszZY1",
				"gap": 12.286918368615488,
				"focus": 0.5010354195317678
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					260.92264577771266,
					71.86862864193597
				],
				[
					472.81986260569374,
					222.2618882297918
				]
			]
		},
		{
			"type": "text",
			"version": 146,
			"versionNonce": 1417668741,
			"isDeleted": false,
			"id": "26l9fjxB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1589.7413560116474,
			"y": 609.867333443277,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 180,
			"height": 25,
			"seed": 531652837,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996457,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "未完成，需阻塞等待",
			"rawText": "未完成，需阻塞等待",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "pYC8J0Ppbf6fQfYIQFi9-",
			"originalText": "未完成，需阻塞等待",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 608,
			"versionNonce": 817927851,
			"isDeleted": false,
			"id": "Zwit6HcoY4nMFXE21fQ4N",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1247.0606894701632,
			"y": 843.0433358505234,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 133.1847516261446,
			"height": 232.09252285407774,
			"seed": 1340412011,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "zKde4LdN"
				}
			],
			"updated": 1693676285067,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "my3x8EDv5uIfv10cYys0w",
				"gap": 11.260614404539638,
				"focus": 0.7454304405289874
			},
			"endBinding": {
				"elementId": "lkyh9LAPYWEAPEXVlxwuk",
				"gap": 10.636996975473721,
				"focus": -0.4824683447548854
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					117.68016581717643,
					-103.26873437585846
				],
				[
					133.1847516261446,
					-232.09252285407774
				]
			]
		},
		{
			"type": "text",
			"version": 110,
			"versionNonce": 869961701,
			"isDeleted": false,
			"id": "zKde4LdN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1370.1742452578865,
			"y": 729.4022023664318,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 40,
			"height": 25,
			"seed": 2043325189,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "自旋",
			"rawText": "自旋",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Zwit6HcoY4nMFXE21fQ4N",
			"originalText": "自旋",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 868,
			"versionNonce": 1980876491,
			"isDeleted": false,
			"id": "eG2-OWdf0NS1ai2QswucL",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 861.2728062991985,
			"y": 896.5628913291534,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 0.3285124082801758,
			"height": 136.37419130402145,
			"seed": 1493568325,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285064,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "mFxwk24xTJuxbLzKcMrU9",
				"gap": 13.479972276605281,
				"focus": -0.021329311128976788
			},
			"endBinding": {
				"elementId": "HPoLvy7-v6RD4ZpgP9BHf",
				"gap": 11.4021379161194,
				"focus": 0.041646108026230325
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-0.3285124082801758,
					136.37419130402145
				]
			]
		},
		{
			"type": "rectangle",
			"version": 935,
			"versionNonce": 1411992389,
			"isDeleted": false,
			"id": "MhiMavhcoqW2m4r_RSvf6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1584.2629036149751,
			"y": 974.4145862964792,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 170,
			"height": 100,
			"seed": 649143787,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "jvppgQQi"
				},
				{
					"id": "mN3EqTUfaXp2NNMo0DCjh",
					"type": "arrow"
				},
				{
					"id": "Q3r5b4om38F81cV2c4mm4",
					"type": "arrow"
				},
				{
					"id": "eBMxZU15QaALZYzS5Vwsc",
					"type": "arrow"
				}
			],
			"updated": 1693330996458,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1172,
			"versionNonce": 1448589195,
			"isDeleted": false,
			"id": "jvppgQQi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1599.2629036149751,
			"y": 999.4145862964792,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140,
			"height": 50,
			"seed": 1561014411,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "将当前节点线程\n置为null",
			"rawText": "将当前节点线程置为null",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "MhiMavhcoqW2m4r_RSvf6",
			"originalText": "将当前节点线程置为null",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "rectangle",
			"version": 961,
			"versionNonce": 633969317,
			"isDeleted": false,
			"id": "Kpg5ONBj1OnJBZOWB9zhf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1588.9766400734588,
			"y": 1158.183056425012,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 170,
			"height": 100,
			"seed": 122496203,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "X6glx9Ay"
				},
				{
					"id": "Q3r5b4om38F81cV2c4mm4",
					"type": "arrow"
				},
				{
					"id": "6dHkJfajnP7l_QPMPEEQw",
					"type": "arrow"
				},
				{
					"id": "h8HHCymHVNCG8aApQzSD8",
					"type": "arrow"
				}
			],
			"updated": 1693330996458,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1291,
			"versionNonce": 690916907,
			"isDeleted": false,
			"id": "X6glx9Ay",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1603.9766400734588,
			"y": 1170.683056425012,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140,
			"height": 75,
			"seed": 193221483,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "从等待栈中遍历\n，开始移除被取\n消的节点",
			"rawText": "从等待栈中遍历，开始移除被取消的节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Kpg5ONBj1OnJBZOWB9zhf",
			"originalText": "从等待栈中遍历，开始移除被取消的节点",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "diamond",
			"version": 1668,
			"versionNonce": 828964357,
			"isDeleted": false,
			"id": "k0q20kix30akPCccnqH6J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1573.1342675161632,
			"y": 1306.543233488535,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 201,
			"height": 170,
			"seed": 371909227,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "JHJRt7Zc"
				},
				{
					"id": "6dHkJfajnP7l_QPMPEEQw",
					"type": "arrow"
				},
				{
					"id": "XcIjeT_V-249jBfY9zlbh",
					"type": "arrow"
				},
				{
					"id": "A_E3bdqhB8XBjvcrM4QHh",
					"type": "arrow"
				}
			],
			"updated": 1693330996458,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2139,
			"versionNonce": 875663563,
			"isDeleted": false,
			"id": "JHJRt7Zc",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1633.8842675161632,
			"y": 1354.043233488535,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 80,
			"height": 75,
			"seed": 521755915,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "队列头节\n点已取消\n等待？",
			"rawText": "队列头节点已取消等待？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "k0q20kix30akPCccnqH6J",
			"originalText": "队列头节点已取消等待？",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1057,
			"versionNonce": 235205989,
			"isDeleted": false,
			"id": "qK6MMdQy-1HNpvgN-2kVh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1394.5395965077616,
			"y": 1538.4794882370136,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 170,
			"height": 100,
			"seed": 480838347,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AlARBGTG"
				},
				{
					"id": "XcIjeT_V-249jBfY9zlbh",
					"type": "arrow"
				},
				{
					"id": "eBMxZU15QaALZYzS5Vwsc",
					"type": "arrow"
				}
			],
			"updated": 1693330996458,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1437,
			"versionNonce": 2029731691,
			"isDeleted": false,
			"id": "AlARBGTG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1400.459609935496,
			"y": 1563.4794882370136,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.15997314453125,
			"height": 50,
			"seed": 95711595,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "基于CAS操作移除\n该头节点",
			"rawText": "基于CAS操作移除该头节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "qK6MMdQy-1HNpvgN-2kVh",
			"originalText": "基于CAS操作移除该头节点",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "diamond",
			"version": 1746,
			"versionNonce": 83297029,
			"isDeleted": false,
			"id": "eUOVGAZv1_tiQc7ZMl7gr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1706.3294198955698,
			"y": 1524.4864901453784,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 201,
			"height": 170,
			"seed": 1531400517,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "lNoJKoPb"
				},
				{
					"id": "A_E3bdqhB8XBjvcrM4QHh",
					"type": "arrow"
				},
				{
					"id": "m7oh2AAG3k0UbkgtLfTCz",
					"type": "arrow"
				},
				{
					"id": "h8HHCymHVNCG8aApQzSD8",
					"type": "arrow"
				}
			],
			"updated": 1693676307641,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2331,
			"versionNonce": 1641421323,
			"isDeleted": false,
			"id": "lNoJKoPb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1767.0794198955698,
			"y": 1571.9864901453784,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 80,
			"height": 75,
			"seed": 1731649701,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "当前节点\n等待线程\n不为空？",
			"rawText": "当前节点等待线程不为空？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "eUOVGAZv1_tiQc7ZMl7gr",
			"originalText": "当前节点等待线程不为空？",
			"lineHeight": 1.25,
			"baseline": 68
		},
		{
			"type": "rectangle",
			"version": 1118,
			"versionNonce": 1639489573,
			"isDeleted": false,
			"id": "OwCNF1G6Cvfm9C1kk98xo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1733.231648812654,
			"y": 1771.512923842148,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 170,
			"height": 100,
			"seed": 1524756427,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "OAqhfD8K"
				},
				{
					"id": "m7oh2AAG3k0UbkgtLfTCz",
					"type": "arrow"
				},
				{
					"id": "h8HHCymHVNCG8aApQzSD8",
					"type": "arrow"
				}
			],
			"updated": 1693330996458,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1500,
			"versionNonce": 733782187,
			"isDeleted": false,
			"id": "OAqhfD8K",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1768.231648812654,
			"y": 1809.012923842148,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 100,
			"height": 25,
			"seed": 1212661355,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "移除该节点",
			"rawText": "移除该节点",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "OwCNF1G6Cvfm9C1kk98xo",
			"originalText": "移除该节点",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 373,
			"versionNonce": 907531531,
			"isDeleted": false,
			"id": "mN3EqTUfaXp2NNMo0DCjh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1662.8495319613446,
			"y": 903.361432109743,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 1.7607731518073706,
			"height": 63.1677368210793,
			"seed": 573992293,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285074,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1OdsebFJjJV_VKSO0aQAk",
				"gap": 9.701646555020261,
				"focus": 0.02302002941826573
			},
			"endBinding": {
				"elementId": "MhiMavhcoqW2m4r_RSvf6",
				"gap": 7.885417365656963,
				"focus": -0.03517694821931485
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1.7607731518073706,
					63.1677368210793
				]
			]
		},
		{
			"type": "arrow",
			"version": 391,
			"versionNonce": 205147691,
			"isDeleted": false,
			"id": "Q3r5b4om38F81cV2c4mm4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1671.6044873550518,
			"y": 1084.195280318597,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 2.8673701673524192,
			"height": 70.1068949019857,
			"seed": 1836787947,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285076,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "MhiMavhcoqW2m4r_RSvf6",
				"gap": 9.780694022117814,
				"focus": 0.0011884092480658416
			},
			"endBinding": {
				"elementId": "Kpg5ONBj1OnJBZOWB9zhf",
				"gap": 3.8808812044292154,
				"focus": 0.031006308474155184
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2.8673701673524192,
					70.1068949019857
				]
			]
		},
		{
			"type": "arrow",
			"version": 388,
			"versionNonce": 1410143051,
			"isDeleted": false,
			"id": "6dHkJfajnP7l_QPMPEEQw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1673.115355397961,
			"y": 1259.183056425012,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 2.0779300130313914,
			"height": 47.08973766525378,
			"seed": 375929637,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693676285078,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Kpg5ONBj1OnJBZOWB9zhf",
				"gap": 1,
				"focus": 0.03568275575009783
			},
			"endBinding": {
				"elementId": "k0q20kix30akPCccnqH6J",
				"gap": 1.213258700878356,
				"focus": 0.05295272311463833
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2.0779300130313914,
					47.08973766525378
				]
			]
		},
		{
			"type": "arrow",
			"version": 395,
			"versionNonce": 1254547211,
			"isDeleted": false,
			"id": "XcIjeT_V-249jBfY9zlbh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1619.4354689371014,
			"y": 1444.9703625690697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 130.0954579767465,
			"height": 85.57479793695757,
			"seed": 37203173,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "zmGJzcEa"
				}
			],
			"updated": 1693676285081,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "k0q20kix30akPCccnqH6J",
				"gap": 10.89321805060949,
				"focus": -0.20912929676237982
			},
			"endBinding": {
				"elementId": "qK6MMdQy-1HNpvgN-2kVh",
				"gap": 7.934327730986297,
				"focus": -0.4861386117885111
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-130.0954579767465,
					85.57479793695757
				]
			]
		},
		{
			"type": "text",
			"version": 109,
			"versionNonce": 1046869573,
			"isDeleted": false,
			"id": "zmGJzcEa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1533.2877643627905,
			"y": 1475.2577615375485,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 42.199951171875,
			"height": 25,
			"seed": 666738155,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "XcIjeT_V-249jBfY9zlbh",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 527,
			"versionNonce": 609011115,
			"isDeleted": false,
			"id": "eBMxZU15QaALZYzS5Vwsc",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1376.7605775666893,
			"y": 1591.8909862186388,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 223.08629005655814,
			"height": 572.5875330989318,
			"seed": 356603339,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "JsJeb6jv"
				}
			],
			"updated": 1693676285081,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qK6MMdQy-1HNpvgN-2kVh",
				"gap": 17.77901894107231,
				"focus": -1.1494754644959042
			},
			"endBinding": {
				"elementId": "MhiMavhcoqW2m4r_RSvf6",
				"gap": 11.842523349389467,
				"focus": 0.8086481751525944
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-27.426487357661813,
					-292.2455466303136
				],
				[
					195.65980269889633,
					-572.5875330989318
				]
			]
		},
		{
			"type": "text",
			"version": 110,
			"versionNonce": 423159205,
			"isDeleted": false,
			"id": "JsJeb6jv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1329.3340902090274,
			"y": 1287.1454395883252,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 40,
			"height": 25,
			"seed": 710232965,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "自旋",
			"rawText": "自旋",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "eBMxZU15QaALZYzS5Vwsc",
			"originalText": "自旋",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 471,
			"versionNonce": 1150392043,
			"isDeleted": false,
			"id": "A_E3bdqhB8XBjvcrM4QHh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1728.6670205256555,
			"y": 1440.8049846319407,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 74.97938432819728,
			"height": 80.76058596838061,
			"seed": 445979659,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "ZKGNosKV"
				}
			],
			"updated": 1693676285083,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "k0q20kix30akPCccnqH6J",
				"gap": 8.251369933134171,
				"focus": -0.09251121323415405
			},
			"endBinding": {
				"elementId": "eUOVGAZv1_tiQc7ZMl7gr",
				"gap": 4.2857125333408135,
				"focus": 0.7805386504032188
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					74.97938432819728,
					80.76058596838061
				]
			]
		},
		{
			"type": "text",
			"version": 108,
			"versionNonce": 783739141,
			"isDeleted": false,
			"id": "ZKGNosKV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1730.1690098041465,
			"y": 1462.5685786327724,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 578112101,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "A_E3bdqhB8XBjvcrM4QHh",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 509,
			"versionNonce": 1972502277,
			"isDeleted": false,
			"id": "m7oh2AAG3k0UbkgtLfTCz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1808.9541104811667,
			"y": 1699.2823618212158,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 0.8895820114071284,
			"height": 64.00277209797878,
			"seed": 308821893,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "RTX8onvM"
				}
			],
			"updated": 1693676294834,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "eUOVGAZv1_tiQc7ZMl7gr",
				"gap": 5.245445187553031,
				"focus": -0.008722460377176014
			},
			"endBinding": {
				"elementId": "OwCNF1G6Cvfm9C1kk98xo",
				"gap": 8.227789922953434,
				"focus": -0.08843742769094633
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0.8895820114071284,
					64.00277209797878
				]
			]
		},
		{
			"id": "RTX8onvM",
			"type": "text",
			"x": 1788.2989259009328,
			"y": 1718.7837478702052,
			"width": 42.199951171875,
			"height": 25,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"seed": 836238635,
			"version": 6,
			"versionNonce": 1469451301,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693676302713,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 20,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 18,
			"containerId": "m7oh2AAG3k0UbkgtLfTCz",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"type": "arrow",
			"version": 640,
			"versionNonce": 770143237,
			"isDeleted": false,
			"id": "h8HHCymHVNCG8aApQzSD8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1914.6332475399686,
			"y": 1608.9080427204997,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 143.79574273012213,
			"height": 417.3483967113318,
			"seed": 420052837,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "VFtWO9BK"
				}
			],
			"updated": 1693676315912,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "eUOVGAZv1_tiQc7ZMl7gr",
				"focus": 1.0732282241263211,
				"gap": 5.158269119256175
			},
			"endBinding": {
				"elementId": "Kpg5ONBj1OnJBZOWB9zhf",
				"focus": -0.9103265835976839,
				"gap": 11.860864736387612
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-22.883842395166084,
					-238.0393137613289
				],
				[
					-143.79574273012213,
					-417.3483967113318
				]
			]
		},
		{
			"type": "text",
			"version": 118,
			"versionNonce": 618570021,
			"isDeleted": false,
			"id": "VFtWO9BK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1866.6894304743923,
			"y": 1345.8687289591708,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 50.11994934082031,
			"height": 50,
			"seed": 853566053,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693676321294,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false\n自旋",
			"rawText": "false\n自旋",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "h8HHCymHVNCG8aApQzSD8",
			"originalText": "false\n自旋",
			"lineHeight": 1.25,
			"baseline": 43
		},
		{
			"type": "ellipse",
			"version": 2315,
			"versionNonce": 66726853,
			"isDeleted": false,
			"id": "FXw2ODmr9g0Cs02j4gv_5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 847.438111088713,
			"y": 274.31467937347145,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 182,
			"height": 140,
			"seed": 828617701,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "tacezu1X"
				},
				{
					"id": "du5CYFh9M0HyS91ANM-Kb",
					"type": "arrow"
				}
			],
			"updated": 1693330996458,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 2242,
			"versionNonce": 1635141797,
			"isDeleted": false,
			"id": "tacezu1X",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 898.5913940007372,
			"y": 331.8172046904131,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 80,
			"height": 25,
			"seed": 216919877,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693411314089,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "返回结果",
			"rawText": "返回结果",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "FXw2ODmr9g0Cs02j4gv_5",
			"originalText": "返回结果",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 394,
			"versionNonce": 1465359691,
			"isDeleted": false,
			"id": "du5CYFh9M0HyS91ANM-Kb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1216.713136965466,
			"y": 106.72795234445093,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffa94d",
			"width": 241.78513926931078,
			"height": 161.85467436127922,
			"seed": 2084746923,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "LbLJk4wl"
				}
			],
			"updated": 1693676285088,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "_zeglng8hhlkfc91gWGTx",
				"gap": 14.804287567218765,
				"focus": 0.7765299190875289
			},
			"endBinding": {
				"elementId": "FXw2ODmr9g0Cs02j4gv_5",
				"gap": 11.088868641405341,
				"focus": -0.5528889660947056
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-241.78513926931078,
					161.85467436127922
				]
			]
		},
		{
			"type": "text",
			"version": 107,
			"versionNonce": 1021380011,
			"isDeleted": false,
			"id": "LbLJk4wl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1068.5309859321371,
			"y": 176.64782240799366,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffa94d",
			"width": 50.11994934082031,
			"height": 25,
			"seed": 1129519115,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "du5CYFh9M0HyS91ANM-Kb",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 419,
			"versionNonce": 1104920197,
			"isDeleted": false,
			"id": "JMBMNq0b",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1857.5372411493474,
			"y": 1167.0983495495138,
			"strokeColor": "#1971c2",
			"backgroundColor": "#eaddd7",
			"width": 640,
			"height": 75,
			"seed": 1679146757,
			"groupIds": [
				"Fo2UP667r4LjAQ57ZbAvx"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693330996458,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "被取消的节点可能由以下原因产生：\n等待节点刚构建完成并加入等待队列中，但尚未阻塞时发现任务已完成，\n此时无需阻塞直接返回结果，因此等待线程为空",
			"rawText": "被取消的节点可能由以下原因产生：\n等待节点刚构建完成并加入等待队列中，但尚未阻塞时发现任务已完成，\n此时无需阻塞直接返回结果，因此等待线程为空",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "被取消的节点可能由以下原因产生：\n等待节点刚构建完成并加入等待队列中，但尚未阻塞时发现任务已完成，\n此时无需阻塞直接返回结果，因此等待线程为空",
			"lineHeight": 1.25,
			"baseline": 68
		}
	],
	"appState": {
		"theme": "light",
		"viewBackgroundColor": "#f5faff",
		"currentItemStrokeColor": "#1e1e1e",
		"currentItemBackgroundColor": "#eaddd7",
		"currentItemFillStyle": "hachure",
		"currentItemStrokeWidth": 1,
		"currentItemStrokeStyle": "solid",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 1,
		"currentItemFontSize": 20,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": -645.9405158285413,
		"scrollY": -682.8758761643901,
		"zoom": {
			"value": 0.9317117933928968
		},
		"currentItemRoundness": "round",
		"gridSize": null,
		"currentStrokeOptions": null,
		"previousGridSize": null,
		"frameRendering": {
			"enabled": true,
			"clip": true,
			"name": true,
			"outline": true
		}
	},
	"files": {}
}
```
%%