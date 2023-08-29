---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Text Elements
set(ThreadLocal<?> key, Object value) ^GC7AahB2

计算当前key所属的hash槽 ^Tjrz95sq

判断计算
的hash槽是否已经
有数据 ^TgMckWqZ

当前槽为空，直接把数据
放置到此为止 ^5fz1GrTr

启发式扫描并清理当前map中的脏数据 ^0G2kiKfZ

未回收脏
数据且长度超过
hash表容量75% ^EWYM5bxy

rehash，新的hash表长度为原来的两倍 ^3FFNqegp

结束 ^w32DwnMJ

从当前槽开始寻找能存放该数据的槽，进行新值插入，遇到空槽时停止 ^KS9auWrS

true ^xpwI8oVY

当前槽的
数据key是否与给定
key相同 ^aLA9nlcM

更新当前槽的数据为value ^EMab5nm1

true ^VJL8qAyN

当前槽的
数据key是否
为null ^XjTRT9XX

false ^Ft2vYUcu

false
使用开放地址放重新计算hash槽 ^5Fhf6okP

认定当前槽是一个脏数据，记为staleSlot，从staleSlot开始清理并更新数据 ^DebJ7p0F

从当前槽开始往前查找脏槽，记为slotToExpunge ^AyW2Y6C4

从staleSlot开始往后查找脏槽 ^upHm9JqX

查找到
key相同的槽 ^lB4IWJ2t

将value更新到槽中，并将槽数据与staleSlot位置互换 ^AKE26X9p

从slotToExpunge开始清理槽中key为null的脏数据 ^Iq4OQuY9

启发式贪心清理其它的脏槽，按比例进行最大努力清理 ^3U6X046w

将value覆盖到
staleSlot槽中 ^OdyXTW0c

slotToExpunge
是否与staleSlot
相等 ^L3nS944u

true ^Zqon31a5

false ^21n3w6UH

true ^vEUtpMHM

false ^GZfGxkwO

true ^IraALWd2

false ^RID7hBEU

ThreadLocal声明 ^gM3Josfy

ThreadLocal对象实例 ^ps6Ed9tQ

Entry ^E3bidyFV

Entry ^P7oNOlNE

Entry ^z8NljqPX

value ^huNKEHX6

key ^B1ytGnPD

JVM 堆对象引用情况 ^CC4klGTp

ThreadLocalMap ^dRAX5KR9

Weak Reference ^bLeyZJ4P

Strongly Reference ^jmRpKkoh

false ^6HeOGlaD

按比例回收，回收次数为
当前哈希表长度处以2，
直到为0。即log(n)的扫描次数 ^pxmNdqXn

按比例回收，回收次数为
当前哈希表长度处以2，
直到为0 ^C2CbM31X

get()方法调用 ^4o9RGpSy

从当前线程中获取ThreadLocalMap对象 ^EbSLFXqj

map
为空？ ^ggZ5eXgo

获取map中该ThreadLocal对应Entry ^xFSBzjWj

Entry
为空？ ^anP1sdEG

返回该值 ^gHGWM3ZB

false ^VezdmPpz

false ^TJ3IpQQy

开始初始化该值 ^ULCzjDV3

true ^RmiAXw8L

true ^I0LZ5LMZ

map
为空？ ^TFXXsDvl

为该线程创建一个ThreadLocalMap实例并初始化该值 ^so00m580

new ThreadLocalMap(this, firstValue); ^IxQqd5fo

在map中设置该ThreadLocal对象对应的值 ^vMFgjDH7

返回初始化的值 ^3xtUbddH

使用开放地址法寻址 ^1bezDJzh

找到了过期的数据？ ^PEtP0Pni

过期数据：key已经被回收
但value未被回收 ^t8JA2zsR

清理当前过期数据 ^xZppoh6a

找到了其他过期的数据？ ^VnPM0VWB

计算未过期数据的哈希槽，是否应位于当前位置？ ^9eXw2dWN

对该元素重新rehash放到合适的位置 ^86i4X2V2

true ^JW58vCOM

false ^zIPMfYVZ

false ^uZNRrXoC

true ^ZMts38zj

true ^o164zHVU

继续循环寻找 ^Pq9K9J1q

往后寻找其他过期数据，直到遇到空槽 ^GAmFZmDr

true ^bobj21M9

false ^QxfCD9js

往后未找到key相同的槽。若往后寻找过程
发现脏槽，且staleSlot和slotToExpunge相等，
则更新slotToExpunge为当前脏槽位置 ^vbIORLCP

若slotToExpunge与staleSlot脏槽相同，
代表staleSlot前无脏槽，更新slotToExpunge为当前槽位置 ^qjdS1HD8

相等代表往前与往后都没有
找到脏槽，无需清理 ^DynALBCf

set(ThreadLocal<?> key, Object value) ^VCyv2XUy

找到空槽？ ^btk8qdn0

false ^KUPH1fNB

true ^hUy6Oc7o

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
			"version": 503,
			"versionNonce": 261059691,
			"isDeleted": false,
			"id": "H7JgZBK3-4ISXgXxlWm3m",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2442.843630041981,
			"y": 1404.0972113760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 129,
			"height": 77,
			"seed": 1019530467,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "gM3Josfy"
				},
				{
					"id": "UGI7mmj8s8OdZpBhMmh-J",
					"type": "arrow"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 503,
			"versionNonce": 411538859,
			"isDeleted": false,
			"id": "gM3Josfy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2435.111604895497,
			"y": 1422.5972113760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 113.53594970703125,
			"height": 40,
			"seed": 6890477,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "ThreadLocal声\n明",
			"rawText": "ThreadLocal声明",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "H7JgZBK3-4ISXgXxlWm3m",
			"originalText": "ThreadLocal声明",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 525,
			"versionNonce": 1912449099,
			"isDeleted": false,
			"id": "aGAMDMZzFxdbZDwhnh487",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2139.8852967086477,
			"y": 1300.6128363760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 753.479166666667,
			"height": 375.58858264777473,
			"seed": 907554275,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 453,
			"versionNonce": 1683515115,
			"isDeleted": false,
			"id": "pAElg1OscOdu7q3CPfO3k",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2099.999880041981,
			"y": 1407.7378363760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 127,
			"height": 72,
			"seed": 623412429,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "ps6Ed9tQ"
				},
				{
					"id": "RDUacjA2CC-7lW0fpnFQv",
					"type": "arrow"
				},
				{
					"id": "UGI7mmj8s8OdZpBhMmh-J",
					"type": "arrow"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 443,
			"versionNonce": 1369287371,
			"isDeleted": false,
			"id": "ps6Ed9tQ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2093.267854895497,
			"y": 1423.7378363760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 113.53594970703125,
			"height": 40,
			"seed": 1226460195,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "ThreadLocal对\n象实例",
			"rawText": "ThreadLocal对象实例",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "pAElg1OscOdu7q3CPfO3k",
			"originalText": "ThreadLocal对象实例",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 692,
			"versionNonce": 97786219,
			"isDeleted": false,
			"id": "Wve9Zl2h3TLB6iHEf4YAP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1795.6925883753145,
			"y": 1411.5086697093684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 127,
			"height": 72,
			"seed": 785925549,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "E3bidyFV"
				},
				{
					"id": "5U-sLAcAsOnAuCWThY0XB",
					"type": "arrow"
				},
				{
					"id": "4JbCfQNEU3UwKKPCiJM1q",
					"type": "arrow"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 422,
			"versionNonce": 300445003,
			"isDeleted": false,
			"id": "E3bidyFV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1753.0325770838106,
			"y": 1437.5086697093684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 41.67997741699219,
			"height": 20,
			"seed": 947576771,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "Entry",
			"rawText": "Entry",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Wve9Zl2h3TLB6iHEf4YAP",
			"originalText": "Entry",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 744,
			"versionNonce": 614632427,
			"isDeleted": false,
			"id": "C9IWsQ5S5kBfX1-33AYT6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1668.6925883753147,
			"y": 1411.5086697093684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 127,
			"height": 72,
			"seed": 1395215523,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "P7oNOlNE"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 416,
			"versionNonce": 601872011,
			"isDeleted": false,
			"id": "P7oNOlNE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1626.0325770838108,
			"y": 1437.5086697093684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 41.67997741699219,
			"height": 20,
			"seed": 1566084941,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "Entry",
			"rawText": "Entry",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "C9IWsQ5S5kBfX1-33AYT6",
			"originalText": "Entry",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 741,
			"versionNonce": 1851444523,
			"isDeleted": false,
			"id": "2YKxP6VP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1541.6925883753152,
			"y": 1411.5086697093684,
			"strokeColor": "#1E1E1EFF",
			"backgroundColor": "#A5D8FFFF",
			"width": 127,
			"height": 72,
			"seed": 1395215523,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "z8NljqPX"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 416,
			"versionNonce": 662564811,
			"isDeleted": false,
			"id": "z8NljqPX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1499.0325770838112,
			"y": 1437.5086697093684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 41.67997741699219,
			"height": 20,
			"seed": 498774851,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "Entry",
			"rawText": "Entry",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "2YKxP6VP",
			"originalText": "Entry",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "ellipse",
			"version": 527,
			"versionNonce": 560184939,
			"isDeleted": false,
			"id": "G8U43p0jnnkjOdRhPrMCA",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1701.9477967086477,
			"y": 1541.9722113760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 85,
			"height": 63,
			"seed": 57895725,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "huNKEHX6"
				},
				{
					"id": "4JbCfQNEU3UwKKPCiJM1q",
					"type": "arrow"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 505,
			"versionNonce": 1162960811,
			"isDeleted": false,
			"id": "huNKEHX6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1680.0438259979433,
			"y": 1563.698347768659,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 41.087982177734375,
			"height": 20,
			"seed": 563576205,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "value",
			"rawText": "value",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "G8U43p0jnnkjOdRhPrMCA",
			"originalText": "value",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "ellipse",
			"version": 523,
			"versionNonce": 922628683,
			"isDeleted": false,
			"id": "Jst0KSo0UhAjsBiGaqcxt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1833.8384217086475,
			"y": 1541.9722113760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 85,
			"height": 63,
			"seed": 642414051,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "B1ytGnPD"
				},
				{
					"id": "5U-sLAcAsOnAuCWThY0XB",
					"type": "arrow"
				},
				{
					"id": "RDUacjA2CC-7lW0fpnFQv",
					"type": "arrow"
				}
			],
			"updated": 1693243573840,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 503,
			"versionNonce": 355119659,
			"isDeleted": false,
			"id": "B1ytGnPD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1803.4144543548766,
			"y": 1563.698347768659,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 24.047988891601562,
			"height": 20,
			"seed": 1709687171,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "key",
			"rawText": "key",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Jst0KSo0UhAjsBiGaqcxt",
			"originalText": "key",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1371,
			"versionNonce": 302991301,
			"isDeleted": false,
			"id": "5U-sLAcAsOnAuCWThY0XB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1731.0936300419808,
			"y": 1492.7274197093686,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 50.102692135124926,
			"height": 46.89378608862489,
			"seed": 1876333037,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243573889,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Wve9Zl2h3TLB6iHEf4YAP",
				"focus": -0.4846049412138607,
				"gap": 9.218750000000227
			},
			"endBinding": {
				"elementId": "Jst0KSo0UhAjsBiGaqcxt",
				"focus": -0.48006512611907237,
				"gap": 3.211223261848115
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
					-50.102692135124926,
					46.89378608862489
				]
			]
		},
		{
			"type": "arrow",
			"version": 1397,
			"versionNonce": 710531717,
			"isDeleted": false,
			"id": "4JbCfQNEU3UwKKPCiJM1q",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1729.4530050419808,
			"y": 1496.5815863760354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 72.90342788408543,
			"height": 40.100270863725996,
			"seed": 2093541731,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243573889,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Wve9Zl2h3TLB6iHEf4YAP",
				"focus": 0.6706244771618252,
				"gap": 13.07291666666697
			},
			"endBinding": {
				"elementId": "G8U43p0jnnkjOdRhPrMCA",
				"focus": 0.9785305201316687,
				"gap": 5.357401365116278
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
					72.90342788408543,
					40.100270863725996
				]
			]
		},
		{
			"type": "arrow",
			"version": 1420,
			"versionNonce": 1840168261,
			"isDeleted": false,
			"id": "RDUacjA2CC-7lW0fpnFQv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1843.055682613441,
			"y": 1562.3819404071742,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 120.53794742854029,
			"height": 112.12022894675624,
			"seed": 1261144611,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243573889,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Jst0KSo0UhAjsBiGaqcxt",
				"focus": -0.7322907994375173,
				"gap": 11.039069902854564
			},
			"endBinding": {
				"elementId": "pAElg1OscOdu7q3CPfO3k",
				"focus": -0.644723842699937,
				"gap": 9.40625
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
					-120.53794742854029,
					-112.12022894675624
				]
			]
		},
		{
			"type": "arrow",
			"version": 1425,
			"versionNonce": 968486917,
			"isDeleted": false,
			"id": "UGI7mmj8s8OdZpBhMmh-J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2305.1300883753142,
			"y": 1445.2899197093686,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 195.27604166666742,
			"height": 0.7004308638647672,
			"seed": 129685293,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243573889,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "H7JgZBK3-4ISXgXxlWm3m",
				"focus": 0.06274246188456858,
				"gap": 8.71354166666697
			},
			"endBinding": {
				"elementId": "pAElg1OscOdu7q3CPfO3k",
				"focus": -0.06943918165455908,
				"gap": 9.854166666665606
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
					195.27604166666742,
					0.7004308638647672
				]
			]
		},
		{
			"type": "text",
			"version": 545,
			"versionNonce": 1178531659,
			"isDeleted": false,
			"id": "CC4klGTp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2119.979046708648,
			"y": 1319.1701280427021,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 150.67198181152344,
			"height": 20,
			"seed": 1903414861,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "JVM 堆对象引用情况",
			"rawText": "JVM 堆对象引用情况",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "JVM 堆对象引用情况",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "text",
			"version": 517,
			"versionNonce": 1637340651,
			"isDeleted": false,
			"id": "dRAX5KR9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1685.2082133753142,
			"y": 1361.5659613760351,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 128.3519287109375,
			"height": 20,
			"seed": 1459311907,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "ThreadLocalMap",
			"rawText": "ThreadLocalMap",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ThreadLocalMap",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "text",
			"version": 496,
			"versionNonce": 1738268811,
			"isDeleted": false,
			"id": "bLeyZJ4P",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1868.8180797909467,
			"y": 1628.7895949611034,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 123.16793823242188,
			"height": 20,
			"seed": 1791371981,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "Weak Reference",
			"rawText": "Weak Reference",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Weak Reference",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "text",
			"version": 557,
			"versionNonce": 839227179,
			"isDeleted": false,
			"id": "jmRpKkoh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1696.8641160647685,
			"y": 1628.7895949611034,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 145.72792053222656,
			"height": 20,
			"seed": 774088131,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243573840,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "Strongly Reference",
			"rawText": "Strongly Reference",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Strongly Reference",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 655,
			"versionNonce": 1580677,
			"isDeleted": false,
			"id": "nzVzk-m7P4EKtjHZsx9xH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -983.5198492167826,
			"y": -281.99458634525894,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 368.7981234303338,
			"height": 4.541565131107916,
			"seed": 1355320557,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242680932,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "dPTtpbMtozWyadMgE4VZT",
				"gap": 10.045494915498239,
				"focus": -0.15047506170708685
			},
			"endBinding": {
				"elementId": "kPCsFHlJo8DVvxcUcuWLY",
				"gap": 9.873802433865535,
				"focus": -0.07041955275244037
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
					368.7981234303338,
					4.541565131107916
				]
			]
		},
		{
			"type": "rectangle",
			"version": 594,
			"versionNonce": 2100562155,
			"isDeleted": false,
			"id": "3ZMH1REJOxCMAMINO_pfk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1878.7346820657983,
			"y": -1471.6296600234068,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 342,
			"height": 91,
			"seed": 1635912867,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "4o9RGpSy"
				},
				{
					"id": "ubW48w_bRcfzgpGLeODeU",
					"type": "arrow"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 547,
			"versionNonce": 647860037,
			"isDeleted": false,
			"id": "4o9RGpSy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1758.6626682108179,
			"y": -1436.1296600234068,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 101.85597229003906,
			"height": 20,
			"seed": 287060963,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "get()方法调用",
			"rawText": "get()方法调用",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "3ZMH1REJOxCMAMINO_pfk",
			"originalText": "get()方法调用",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 548,
			"versionNonce": 1194963851,
			"isDeleted": false,
			"id": "rrFtUuUP6_mP8M8MIZmpv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1827.490496315547,
			"y": -1311.5312267289432,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 221,
			"height": 76,
			"seed": 1838152013,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "EbSLFXqj"
				},
				{
					"id": "ubW48w_bRcfzgpGLeODeU",
					"type": "arrow"
				},
				{
					"id": "ey02iPLor1z17jcG9oMyd",
					"type": "arrow"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 544,
			"versionNonce": 689266341,
			"isDeleted": false,
			"id": "EbSLFXqj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1822.318476357051,
			"y": -1293.5312267289432,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 210.6559600830078,
			"height": 40,
			"seed": 1615921667,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "从当前线程中获取ThreadLoc\nalMap对象",
			"rawText": "从当前线程中获取ThreadLocalMap对象",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "rrFtUuUP6_mP8M8MIZmpv",
			"originalText": "从当前线程中获取ThreadLocalMap对象",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "diamond",
			"version": 637,
			"versionNonce": 611387947,
			"isDeleted": false,
			"id": "F3rDIQBQ90d_UwRkiO7fb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1776.992017257334,
			"y": -1168.5766135238282,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 126,
			"height": 143,
			"seed": 1851961571,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "ggZ5eXgo"
				},
				{
					"id": "ey02iPLor1z17jcG9oMyd",
					"type": "arrow"
				},
				{
					"id": "qVo6qizrTwtBC0ATga6Rv",
					"type": "arrow"
				},
				{
					"id": "ZLC0KhN3UaUIIhUvPuS-a",
					"type": "arrow"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 580,
			"versionNonce": 1587298821,
			"isDeleted": false,
			"id": "ggZ5eXgo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1737.992017257334,
			"y": -1116.8266135238282,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 48,
			"height": 40,
			"seed": 1118635501,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "map\n为空？",
			"rawText": "map\n为空？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "F3rDIQBQ90d_UwRkiO7fb",
			"originalText": "map\n为空？",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 620,
			"versionNonce": 432311499,
			"isDeleted": false,
			"id": "4oCzCd9Krg_X3lI8yMOl0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1804.9480656828016,
			"y": -959.125223839718,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 166,
			"height": 98,
			"seed": 338026755,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "xFSBzjWj"
				},
				{
					"id": "qVo6qizrTwtBC0ATga6Rv",
					"type": "arrow"
				},
				{
					"id": "r87qwyjW46bnkcpVEsn43",
					"type": "arrow"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 643,
			"versionNonce": 1009760613,
			"isDeleted": false,
			"id": "xFSBzjWj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1796.3800343717664,
			"y": -930.125223839718,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 148.8639373779297,
			"height": 40,
			"seed": 1575446115,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "获取map中该Thread\nLocal对应Entry",
			"rawText": "获取map中该ThreadLocal对应Entry",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "4oCzCd9Krg_X3lI8yMOl0",
			"originalText": "获取map中该ThreadLocal对应Entry",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "diamond",
			"version": 645,
			"versionNonce": 718191467,
			"isDeleted": false,
			"id": "dYoxeWgFOqE8_Yj-yB6ey",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1830.5635832322498,
			"y": -783.1355691521908,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 126,
			"height": 143,
			"seed": 883679917,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "anP1sdEG"
				},
				{
					"id": "r87qwyjW46bnkcpVEsn43",
					"type": "arrow"
				},
				{
					"id": "-pjGzbh_m4zg0UDmFRUyz",
					"type": "arrow"
				},
				{
					"id": "3mID0RnAa0KP0QnjcQHRS",
					"type": "arrow"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 600,
			"versionNonce": 991773893,
			"isDeleted": false,
			"id": "anP1sdEG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1791.5635832322498,
			"y": -731.3855691521908,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 48,
			"height": 40,
			"seed": 332727565,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "Entry\n为空？",
			"rawText": "Entry\n为空？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "dYoxeWgFOqE8_Yj-yB6ey",
			"originalText": "Entry\n为空？",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 687,
			"versionNonce": 64624139,
			"isDeleted": false,
			"id": "QHkHXPktpNrKwv2wMaOMS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1856.3421647225853,
			"y": -542.9224720431392,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 166,
			"height": 98,
			"seed": 625150829,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "gHGWM3ZB"
				},
				{
					"id": "-pjGzbh_m4zg0UDmFRUyz",
					"type": "arrow"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 726,
			"versionNonce": 1325556773,
			"isDeleted": false,
			"id": "gHGWM3ZB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1805.3421647225853,
			"y": -503.92247204313924,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 64,
			"height": 20,
			"seed": 1627985357,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "返回该值",
			"rawText": "返回该值",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "QHkHXPktpNrKwv2wMaOMS",
			"originalText": "返回该值",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1545,
			"versionNonce": 1675697323,
			"isDeleted": false,
			"id": "ubW48w_bRcfzgpGLeODeU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1708.0895490271953,
			"y": -1371.1875740954392,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 1.444645372584091,
			"height": 56.618339863898655,
			"seed": 327187725,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "3ZMH1REJOxCMAMINO_pfk",
				"gap": 9.442085927967582,
				"focus": 0.01020406670910639
			},
			"endBinding": {
				"elementId": "rrFtUuUP6_mP8M8MIZmpv",
				"gap": 3.0380075025973383,
				"focus": 0.10220453958332616
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
					1.444645372584091,
					56.618339863898655
				]
			]
		},
		{
			"type": "arrow",
			"version": 1577,
			"versionNonce": 466392965,
			"isDeleted": false,
			"id": "ey02iPLor1z17jcG9oMyd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1714.7834106892856,
			"y": -1227.3277721092568,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 1.2820677727449947,
			"height": 53.68017979510205,
			"seed": 2097284835,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "rrFtUuUP6_mP8M8MIZmpv",
				"gap": 8.203454619686454,
				"focus": -0.009905860247955152
			},
			"endBinding": {
				"elementId": "F3rDIQBQ90d_UwRkiO7fb",
				"gap": 5.0946626189232305,
				"focus": 0.036816719313423514
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
					1.2820677727449947,
					53.68017979510205
				]
			]
		},
		{
			"type": "arrow",
			"version": 1515,
			"versionNonce": 1183731531,
			"isDeleted": false,
			"id": "qVo6qizrTwtBC0ATga6Rv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1715.8111779558922,
			"y": -1022.0001870367776,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 1.027545428989015,
			"height": 54.50700356860784,
			"seed": 733890979,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "VezdmPpz"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "F3rDIQBQ90d_UwRkiO7fb",
				"gap": 4.0125019706482306,
				"focus": 0.05134084307424681
			},
			"endBinding": {
				"elementId": "4oCzCd9Krg_X3lI8yMOl0",
				"gap": 8.367959628451672,
				"focus": 0.09825483565049963
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
					1.027545428989015,
					54.50700356860784
				]
			]
		},
		{
			"type": "text",
			"version": 484,
			"versionNonce": 1807310565,
			"isDeleted": false,
			"id": "VezdmPpz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1735.3355598493138,
			"y": -1004.2254221867313,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 40.09596252441406,
			"height": 20,
			"seed": 535233645,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "qVo6qizrTwtBC0ATga6Rv",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1696,
			"versionNonce": 1312566763,
			"isDeleted": false,
			"id": "r87qwyjW46bnkcpVEsn43",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1722.9080807893401,
			"y": -848.5913101135814,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 44.32383797827106,
			"height": 59.39205836179667,
			"seed": 215319523,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "4oCzCd9Krg_X3lI8yMOl0",
				"gap": 12.533913726136689,
				"focus": -0.3760383305154643
			},
			"endBinding": {
				"elementId": "dYoxeWgFOqE8_Yj-yB6ey",
				"gap": 6.072746329768933,
				"focus": -0.9135478954932242
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
					-44.32383797827106,
					59.39205836179667
				]
			]
		},
		{
			"type": "arrow",
			"version": 1731,
			"versionNonce": 1700149829,
			"isDeleted": false,
			"id": "-pjGzbh_m4zg0UDmFRUyz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1768.3333793072252,
			"y": -637.1749770633262,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 2.312416402437293,
			"height": 86.8987368378954,
			"seed": 1703442595,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "TJ3IpQQy"
				}
			],
			"updated": 1693242557292,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "dYoxeWgFOqE8_Yj-yB6ey",
				"gap": 3.0590344087137424,
				"focus": -0.019232308333846017
			},
			"endBinding": {
				"elementId": "QHkHXPktpNrKwv2wMaOMS",
				"gap": 7.35376818229156,
				"focus": 0.014195884851369644
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
					-2.312416402437293,
					86.8987368378954
				]
			]
		},
		{
			"type": "text",
			"version": 484,
			"versionNonce": 1844787339,
			"isDeleted": false,
			"id": "TJ3IpQQy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1788.2846315337026,
			"y": -619.9883947235097,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 40.09596252441406,
			"height": 20,
			"seed": 1621916515,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "-pjGzbh_m4zg0UDmFRUyz",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 779,
			"versionNonce": 25209253,
			"isDeleted": false,
			"id": "UY2G1O66J1bA-Z8pxIcTc",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1398.5701622849444,
			"y": -760.6355691521907,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 166,
			"height": 98,
			"seed": 85440685,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "ULCzjDV3"
				},
				{
					"id": "ZLC0KhN3UaUIIhUvPuS-a",
					"type": "arrow"
				},
				{
					"id": "3mID0RnAa0KP0QnjcQHRS",
					"type": "arrow"
				},
				{
					"id": "urZD-SopW_l7tUwgUEhbq",
					"type": "arrow"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 824,
			"versionNonce": 1400106795,
			"isDeleted": false,
			"id": "ULCzjDV3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1371.5701622849444,
			"y": -721.6355691521907,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 112,
			"height": 20,
			"seed": 1414929165,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "开始初始化该值",
			"rawText": "开始初始化该值",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "UY2G1O66J1bA-Z8pxIcTc",
			"originalText": "开始初始化该值",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1519,
			"versionNonce": 1097215237,
			"isDeleted": false,
			"id": "ZLC0KhN3UaUIIhUvPuS-a",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1641.9161601213837,
			"y": -1101.0522983809017,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 330.8857119287743,
			"height": 331.3327462397193,
			"seed": 926551181,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "RmiAXw8L"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "F3rDIQBQ90d_UwRkiO7fb",
				"gap": 9.908443512224622,
				"focus": -0.735011887495454
			},
			"endBinding": {
				"elementId": "UY2G1O66J1bA-Z8pxIcTc",
				"gap": 9.083982988991693,
				"focus": 0.36447722231123286
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
					209.2373325460751,
					141.02171615425016
				],
				[
					330.8857119287743,
					331.3327462397193
				]
			]
		},
		{
			"type": "text",
			"version": 483,
			"versionNonce": 287494603,
			"isDeleted": false,
			"id": "RmiAXw8L",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1449.558817199332,
			"y": -970.0305822266516,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 33.759979248046875,
			"height": 20,
			"seed": 1716560749,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ZLC0KhN3UaUIIhUvPuS-a",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1557,
			"versionNonce": 612648037,
			"isDeleted": false,
			"id": "3mID0RnAa0KP0QnjcQHRS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1694.8816683984596,
			"y": -712.0262333630111,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 286.3586553089201,
			"height": 1.9778822721377765,
			"seed": 1472355523,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "I0LZ5LMZ"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "dYoxeWgFOqE8_Yj-yB6ey",
				"gap": 9.689793257566414,
				"focus": 0.0015573503008604731
			},
			"endBinding": {
				"elementId": "UY2G1O66J1bA-Z8pxIcTc",
				"gap": 9.952850804595073,
				"focus": 0.06072974586294891
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
					286.3586553089201,
					-1.9778822721377765
				]
			]
		},
		{
			"type": "text",
			"version": 483,
			"versionNonce": 136077419,
			"isDeleted": false,
			"id": "I0LZ5LMZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1565.2459346448702,
			"y": -723.0382190197582,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 33.759979248046875,
			"height": 20,
			"seed": 1528814179,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "3mID0RnAa0KP0QnjcQHRS",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "diamond",
			"version": 711,
			"versionNonce": 1588096965,
			"isDeleted": false,
			"id": "LL4gtf1mfZAdNVCib73i2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1385.1046628655893,
			"y": -579.163401284697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 126,
			"height": 143,
			"seed": 704832781,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "TFXXsDvl"
				},
				{
					"id": "urZD-SopW_l7tUwgUEhbq",
					"type": "arrow"
				},
				{
					"id": "FOTVwoYx1mYSRgjRCqryH",
					"type": "arrow"
				},
				{
					"id": "048ebS9AuEaZk_nxxcD-d",
					"type": "arrow"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 651,
			"versionNonce": 304814859,
			"isDeleted": false,
			"id": "TFXXsDvl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1346.1046628655893,
			"y": -527.413401284697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 48,
			"height": 40,
			"seed": 534559597,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "map\n为空？",
			"rawText": "map\n为空？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "LL4gtf1mfZAdNVCib73i2",
			"originalText": "map\n为空？",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 932,
			"versionNonce": 1307586341,
			"isDeleted": false,
			"id": "HpNX41pI9y-IwCcb_EgX9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1570.9533112786637,
			"y": -324.6151385380979,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 166,
			"height": 98,
			"seed": 591999971,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "so00m580"
				},
				{
					"id": "FOTVwoYx1mYSRgjRCqryH",
					"type": "arrow"
				},
				{
					"id": "eaDWYep_WO-SrA58Rs3pw",
					"type": "arrow"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1095,
			"versionNonce": 1808368043,
			"isDeleted": false,
			"id": "so00m580",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1565.7773011468278,
			"y": -305.6151385380979,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 155.64797973632812,
			"height": 60,
			"seed": 274644867,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "为该线程创建一个Thr\neadLocalMap实例并\n初始化该值",
			"rawText": "为该线程创建一个ThreadLocalMap实例并初始化该值",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "HpNX41pI9y-IwCcb_EgX9",
			"originalText": "为该线程创建一个ThreadLocalMap实例并初始化该值",
			"lineHeight": 1.25,
			"baseline": 54
		},
		{
			"type": "text",
			"version": 692,
			"versionNonce": 818479749,
			"isDeleted": false,
			"id": "IxQqd5fo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1885.961510526633,
			"y": -283.2982161809547,
			"strokeColor": "#1971c2",
			"backgroundColor": "#eaddd7",
			"width": 296.623779296875,
			"height": 20,
			"seed": 1458372877,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "new ThreadLocalMap(this, firstValue);",
			"rawText": "new ThreadLocalMap(this, firstValue);",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "new ThreadLocalMap(this, firstValue);",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 1038,
			"versionNonce": 1579291723,
			"isDeleted": false,
			"id": "dPTtpbMtozWyadMgE4VZT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1159.5653441322809,
			"y": -324.6151385380979,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 166,
			"height": 98,
			"seed": 708329997,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "vMFgjDH7"
				},
				{
					"id": "048ebS9AuEaZk_nxxcD-d",
					"type": "arrow"
				},
				{
					"id": "PYVheSZQcMZH9AOK1majW",
					"type": "arrow"
				},
				{
					"id": "nzVzk-m7P4EKtjHZsx9xH",
					"type": "arrow"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1295,
			"versionNonce": 468740581,
			"isDeleted": false,
			"id": "vMFgjDH7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1154.4453184975152,
			"y": -295.6151385380979,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 155.75994873046875,
			"height": 40,
			"seed": 190740589,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "在map中设置该Threa\ndLocal对象对应的值",
			"rawText": "在map中设置该ThreadLocal对象对应的值",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "dPTtpbMtozWyadMgE4VZT",
			"originalText": "在map中设置该ThreadLocal对象对应的值",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "arrow",
			"version": 1542,
			"versionNonce": 688870123,
			"isDeleted": false,
			"id": "urZD-SopW_l7tUwgUEhbq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1315.53674350094,
			"y": -653.0098441484805,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 0.36012743018022775,
			"height": 74.44972281003709,
			"seed": 881238979,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "UY2G1O66J1bA-Z8pxIcTc",
				"gap": 9.625725003710158,
				"focus": 0.003005453108968582
			},
			"endBinding": {
				"elementId": "LL4gtf1mfZAdNVCib73i2",
				"gap": 4.799264015861304,
				"focus": 0.11541250280597826
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
					0.36012743018022775,
					74.44972281003709
				]
			]
		},
		{
			"type": "arrow",
			"version": 1506,
			"versionNonce": 812901701,
			"isDeleted": false,
			"id": "FOTVwoYx1mYSRgjRCqryH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1357.2591001153057,
			"y": -457.92812469308205,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 129.68082290280813,
			"height": 121.37120904958715,
			"seed": 1219059043,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "LL4gtf1mfZAdNVCib73i2",
				"gap": 11.987592033461361,
				"focus": -0.23543280187853996
			},
			"endBinding": {
				"elementId": "HpNX41pI9y-IwCcb_EgX9",
				"gap": 11.941777105397023,
				"focus": -0.47357577422707364
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
					-129.68082290280813,
					121.37120904958715
				]
			]
		},
		{
			"type": "arrow",
			"version": 1679,
			"versionNonce": 756792715,
			"isDeleted": false,
			"id": "048ebS9AuEaZk_nxxcD-d",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1277.2889052056244,
			"y": -468.69784413775085,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 208.01954545825015,
			"height": 131.3724107524455,
			"seed": 201246339,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "LL4gtf1mfZAdNVCib73i2",
				"gap": 12.116581045043752,
				"focus": 0.14912822458410996
			},
			"endBinding": {
				"elementId": "dPTtpbMtozWyadMgE4VZT",
				"gap": 12.710294847207479,
				"focus": 0.6539093224374548
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
					208.01954545825015,
					131.3724107524455
				]
			]
		},
		{
			"type": "rectangle",
			"version": 826,
			"versionNonce": 296809637,
			"isDeleted": false,
			"id": "L0jzyDDYXNrZRFXBziIRd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1377.354033150364,
			"y": -71.03997991979247,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 166,
			"height": 98,
			"seed": 647776109,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "3xtUbddH"
				},
				{
					"id": "eaDWYep_WO-SrA58Rs3pw",
					"type": "arrow"
				},
				{
					"id": "PYVheSZQcMZH9AOK1majW",
					"type": "arrow"
				}
			],
			"updated": 1693242557293,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 896,
			"versionNonce": 1434571819,
			"isDeleted": false,
			"id": "3xtUbddH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1350.354033150364,
			"y": -32.03997991979247,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 112,
			"height": 20,
			"seed": 1541067213,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "返回初始化的值",
			"rawText": "返回初始化的值",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "L0jzyDDYXNrZRFXBziIRd",
			"originalText": "返回初始化的值",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1666,
			"versionNonce": 458561541,
			"isDeleted": false,
			"id": "eaDWYep_WO-SrA58Rs3pw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1504.3295960931455,
			"y": -216.6261524159902,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 221.7999046766745,
			"height": 133.4743135954817,
			"seed": 1455901123,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "HpNX41pI9y-IwCcb_EgX9",
				"gap": 9.98898612210769,
				"focus": 0.6957610913801201
			},
			"endBinding": {
				"elementId": "L0jzyDDYXNrZRFXBziIRd",
				"gap": 12.111858900716015,
				"focus": 0.6895316651121568
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
					221.7999046766745,
					133.4743135954817
				]
			]
		},
		{
			"type": "arrow",
			"version": 1673,
			"versionNonce": 1110185675,
			"isDeleted": false,
			"id": "PYVheSZQcMZH9AOK1majW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1069.8661631623675,
			"y": -212.63657943937665,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"width": 218.2496716982257,
			"height": 129.056386467695,
			"seed": 1117690883,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "dPTtpbMtozWyadMgE4VZT",
				"gap": 13.978559098721234,
				"focus": -0.6825040006419141
			},
			"endBinding": {
				"elementId": "L0jzyDDYXNrZRFXBziIRd",
				"gap": 12.540213051889168,
				"focus": -0.5898394234110513
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
					-218.2496716982257,
					129.056386467695
				]
			]
		},
		{
			"type": "text",
			"version": 34,
			"versionNonce": 1465765733,
			"isDeleted": false,
			"id": "VCyv2XUy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1046.9111377361555,
			"y": -361.1259559988274,
			"strokeColor": "#1971c2",
			"backgroundColor": "#ffc9c9",
			"width": 305.0238342285156,
			"height": 20,
			"seed": 264141709,
			"groupIds": [
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242557293,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "set(ThreadLocal<?> key, Object value)",
			"rawText": "set(ThreadLocal<?> key, Object value)",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "set(ThreadLocal<?> key, Object value)",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "text",
			"version": 922,
			"versionNonce": 1157161957,
			"isDeleted": false,
			"id": "pxmNdqXn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -832.541992317666,
			"y": 561.3769278276311,
			"strokeColor": "#1971c2",
			"backgroundColor": "#fcc2d7",
			"width": 211.615966796875,
			"height": 60,
			"seed": 1250806509,
			"groupIds": [
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0。即log(n)的扫描次数",
			"rawText": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0。即log(n)的扫描次数",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0。即log(n)的扫描次数",
			"lineHeight": 1.25,
			"baseline": 54
		},
		{
			"type": "rectangle",
			"version": 777,
			"versionNonce": 1315141867,
			"isDeleted": false,
			"id": "kPCsFHlJo8DVvxcUcuWLY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -604.8479233525833,
			"y": -308.10758979080447,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 327,
			"height": 61,
			"seed": 826969101,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "GC7AahB2"
				},
				{
					"id": "4ofT9kvGoGvogoTR78YrG",
					"type": "arrow"
				},
				{
					"id": "nzVzk-m7P4EKtjHZsx9xH",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1415,
			"versionNonce": 1448943429,
			"isDeleted": false,
			"id": "GC7AahB2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -593.8598404668411,
			"y": -287.2075897908045,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 305.0238342285156,
			"height": 19.2,
			"seed": 495543395,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "set(ThreadLocal<?> key, Object value)",
			"rawText": "set(ThreadLocal<?> key, Object value)",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "kPCsFHlJo8DVvxcUcuWLY",
			"originalText": "set(ThreadLocal<?> key, Object value)",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 447,
			"versionNonce": 553329547,
			"isDeleted": false,
			"id": "EINmIFJwSEw6J5Ev_asxp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -526.3224617877219,
			"y": -142.7438507745411,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 183,
			"height": 77,
			"seed": 1056917891,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Tjrz95sq"
				},
				{
					"id": "4ofT9kvGoGvogoTR78YrG",
					"type": "arrow"
				},
				{
					"id": "oEJv1sJIZhPpVnOeyKwXH",
					"type": "arrow"
				},
				{
					"id": "YWlIUUhZ08REye1Z-HVgl",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 413,
			"versionNonce": 1516433061,
			"isDeleted": false,
			"id": "Tjrz95sq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -520.4304406085226,
			"y": -123.44385077454109,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 171.21595764160156,
			"height": 38.4,
			"seed": 1123155085,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "计算当前key所属的hash\n槽",
			"rawText": "计算当前key所属的hash槽",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "EINmIFJwSEw6J5Ev_asxp",
			"originalText": "计算当前key所属的hash槽",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "diamond",
			"version": 921,
			"versionNonce": 612661803,
			"isDeleted": false,
			"id": "0oide05HgsqhqlWv_nDj4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -556.1670480686603,
			"y": 38.84686117223731,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 455658413,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "TgMckWqZ"
				},
				{
					"id": "oEJv1sJIZhPpVnOeyKwXH",
					"type": "arrow"
				},
				{
					"id": "YWlIUUhZ08REye1Z-HVgl",
					"type": "arrow"
				},
				{
					"id": "GPJdiyWt3gOVvOfzEnBOe",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 993,
			"versionNonce": 800576005,
			"isDeleted": false,
			"id": "TgMckWqZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -490.7510324436603,
			"y": 103.4468611722373,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 99.16796875,
			"height": 76.8,
			"seed": 361831459,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "判断计算\n的hash槽是否\n已经\n有数据",
			"rawText": "判断计算\n的hash槽是否已经\n有数据",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "0oide05HgsqhqlWv_nDj4",
			"originalText": "判断计算\n的hash槽是否已经\n有数据",
			"lineHeight": 1.2,
			"baseline": 71
		},
		{
			"type": "arrow",
			"version": 1484,
			"versionNonce": 636091595,
			"isDeleted": false,
			"id": "4ofT9kvGoGvogoTR78YrG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -443.443126243566,
			"y": -238.17472984849067,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 2.069205492593028,
			"height": 86.78506287329171,
			"seed": 1581054733,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "kPCsFHlJo8DVvxcUcuWLY",
				"focus": 0.007033000908736418,
				"gap": 8.932859942313797
			},
			"endBinding": {
				"elementId": "EINmIFJwSEw6J5Ev_asxp",
				"focus": -0.1278318952665658,
				"gap": 8.645816200657862
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
					-2.069205492593028,
					86.78506287329171
				]
			]
		},
		{
			"type": "arrow",
			"version": 1980,
			"versionNonce": 56183141,
			"isDeleted": false,
			"id": "oEJv1sJIZhPpVnOeyKwXH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -443.5275239087672,
			"y": -61.71793523352764,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 0.7651741386217168,
			"height": 97.98029631861208,
			"seed": 27142829,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "EINmIFJwSEw6J5Ev_asxp",
				"focus": 0.0984433718899713,
				"gap": 4.025915541013461
			},
			"endBinding": {
				"elementId": "0oide05HgsqhqlWv_nDj4",
				"focus": -0.006702111002040121,
				"gap": 2.9895432592519597
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
					0.7651741386217168,
					97.98029631861208
				]
			]
		},
		{
			"type": "rectangle",
			"version": 895,
			"versionNonce": 1892035435,
			"isDeleted": false,
			"id": "Ln77prd8BvzhNPGPbMSNP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -540.1865370857865,
			"y": 339.1699140845317,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 214,
			"height": 111,
			"seed": 1004126947,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "5fz1GrTr"
				},
				{
					"id": "GPJdiyWt3gOVvOfzEnBOe",
					"type": "arrow"
				},
				{
					"id": "meWBtmUsoCkato1C6_gIt",
					"type": "arrow"
				},
				{
					"id": "5F-vZ-l_CVHW3xe_7bq_7",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 741,
			"versionNonce": 110417093,
			"isDeleted": false,
			"id": "5fz1GrTr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -521.1865370857865,
			"y": 375.4699140845317,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 176,
			"height": 38.4,
			"seed": 1003113389,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "当前槽为空，直接把数据\n放置到此为止",
			"rawText": "当前槽为空，直接把数据\n放置到此为止",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Ln77prd8BvzhNPGPbMSNP",
			"originalText": "当前槽为空，直接把数据\n放置到此为止",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "rectangle",
			"version": 889,
			"versionNonce": 2036565515,
			"isDeleted": false,
			"id": "EtmeMz-BdnDUhKY0TuO1E",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -540.7665124147338,
			"y": 531.7794946766369,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 201,
			"height": 106,
			"seed": 86109901,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "0G2kiKfZ"
				},
				{
					"id": "meWBtmUsoCkato1C6_gIt",
					"type": "arrow"
				},
				{
					"id": "XvLzY_87WT1etuKK5faAW",
					"type": "arrow"
				},
				{
					"id": "ImqDWpKayT1X0uRagq8S7",
					"type": "arrow"
				},
				{
					"id": "gMQlwVkn7u7qVtj-kBHea",
					"type": "arrow"
				},
				{
					"id": "1vXz6kIEeCJWzvTstvh13",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 850,
			"versionNonce": 1695020069,
			"isDeleted": false,
			"id": "0G2kiKfZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -534.6104989259643,
			"y": 565.5794946766368,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 188.68797302246094,
			"height": 38.4,
			"seed": 293151875,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "启发式扫描并清理当前map\n中的脏数据",
			"rawText": "启发式扫描并清理当前map中的脏数据",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "EtmeMz-BdnDUhKY0TuO1E",
			"originalText": "启发式扫描并清理当前map中的脏数据",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "arrow",
			"version": 2231,
			"versionNonce": 1186761899,
			"isDeleted": false,
			"id": "GPJdiyWt3gOVvOfzEnBOe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -435.20097445293777,
			"y": 242.01029341502837,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 2.015061401333753,
			"height": 93.63591354573535,
			"seed": 197285091,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "6HeOGlaD"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0oide05HgsqhqlWv_nDj4",
				"focus": -0.07705013525274838,
				"gap": 1.8674398074690686
			},
			"endBinding": {
				"elementId": "Ln77prd8BvzhNPGPbMSNP",
				"focus": 0.011699312965970064,
				"gap": 3.523707123767963
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
					-1.4454390722114567,
					50.01573281521044
				],
				[
					0.5696223291222964,
					93.63591354573535
				]
			]
		},
		{
			"type": "text",
			"version": 229,
			"versionNonce": 2057316229,
			"isDeleted": false,
			"id": "6HeOGlaD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -333.7621906035124,
			"y": 287.80046080507276,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 40.09596252441406,
			"height": 20,
			"seed": 480528813,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "GPJdiyWt3gOVvOfzEnBOe",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2621,
			"versionNonce": 477871947,
			"isDeleted": false,
			"id": "meWBtmUsoCkato1C6_gIt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -434.2040438159486,
			"y": 457.14318711084763,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 2.4411182759521353,
			"height": 63.724343692795514,
			"seed": 2024817731,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Ln77prd8BvzhNPGPbMSNP",
				"focus": -0.012606394286405113,
				"gap": 6.973273026315951
			},
			"endBinding": {
				"elementId": "EtmeMz-BdnDUhKY0TuO1E",
				"focus": 0.011440957849532411,
				"gap": 10.911963872993738
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
					-2.4411182759521353,
					63.724343692795514
				]
			]
		},
		{
			"type": "diamond",
			"version": 1037,
			"versionNonce": 1234890469,
			"isDeleted": false,
			"id": "T3l9HpdHo4Bku8vxH8bYJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -538.9767196303102,
			"y": 2071.8489001315897,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 181,
			"height": 212,
			"seed": 1742662403,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "EWYM5bxy"
				},
				{
					"id": "AYCf8n2KnLW4XAKa3FQ-R",
					"type": "arrow"
				},
				{
					"id": "xJj1W6f_5J-goSJcBq56-",
					"type": "arrow"
				},
				{
					"id": "1vXz6kIEeCJWzvTstvh13",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 966,
			"versionNonce": 1694211563,
			"isDeleted": false,
			"id": "EWYM5bxy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -488.2267196303102,
			"y": 2129.8489001315897,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 80,
			"height": 96,
			"seed": 327694115,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "未回收脏\n数据且长度\n超过\nhash表容\n量75%",
			"rawText": "未回收脏\n数据且长度超过\nhash表容量75%",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "T3l9HpdHo4Bku8vxH8bYJ",
			"originalText": "未回收脏\n数据且长度超过\nhash表容量75%",
			"lineHeight": 1.2,
			"baseline": 90
		},
		{
			"type": "rectangle",
			"version": 1026,
			"versionNonce": 814074437,
			"isDeleted": false,
			"id": "1LDn5lFf9YzfJmTwbovMv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -580.6883938814204,
			"y": 2427.663547040864,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 247,
			"height": 118,
			"seed": 1806852419,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "3FFNqegp"
				},
				{
					"id": "AYCf8n2KnLW4XAKa3FQ-R",
					"type": "arrow"
				},
				{
					"id": "PjGHh6io4PNZffjmIAfaD",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 984,
			"versionNonce": 1531357323,
			"isDeleted": false,
			"id": "3FFNqegp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -572.1723568941158,
			"y": 2467.4635470408643,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 229.96792602539062,
			"height": 38.4,
			"seed": 882174541,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "rehash，新的hash表长度为原来\n的两倍",
			"rawText": "rehash，新的hash表长度为原来的两倍",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "1LDn5lFf9YzfJmTwbovMv",
			"originalText": "rehash，新的hash表长度为原来的两倍",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "ellipse",
			"version": 1209,
			"versionNonce": 1614690725,
			"isDeleted": false,
			"id": "cEJ-yBHYHnZKZPLnmew3V",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -538.3824057879126,
			"y": 2778.7604507446126,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffa94d",
			"width": 156,
			"height": 125,
			"seed": 1137199075,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "w32DwnMJ"
				},
				{
					"id": "PjGHh6io4PNZffjmIAfaD",
					"type": "arrow"
				},
				{
					"id": "-ghrUwKzysCBvFwQM59lR",
					"type": "arrow"
				},
				{
					"id": "gWYpx7rL6q0SJw2hdApu2",
					"type": "arrow"
				},
				{
					"id": "xJj1W6f_5J-goSJcBq56-",
					"type": "arrow"
				},
				{
					"id": "4XIY0Qdaz4mUGbX4I3hHt",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1147,
			"versionNonce": 1964465963,
			"isDeleted": false,
			"id": "w32DwnMJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -476.5367347204633,
			"y": 2831.4662769204538,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 32,
			"height": 19.2,
			"seed": 70266371,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "结束",
			"rawText": "结束",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "cEJ-yBHYHnZKZPLnmew3V",
			"originalText": "结束",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 3308,
			"versionNonce": 301055237,
			"isDeleted": false,
			"id": "AYCf8n2KnLW4XAKa3FQ-R",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -449.74075857873777,
			"y": 2286.612535632788,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.5095295634597505,
			"height": 132.37456350150978,
			"seed": 1683993069,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "bobj21M9"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "T3l9HpdHo4Bku8vxH8bYJ",
				"focus": 0.000262496461105659,
				"gap": 2.7557929721055814
			},
			"endBinding": {
				"elementId": "1LDn5lFf9YzfJmTwbovMv",
				"focus": 0.04160620597393211,
				"gap": 8.676447906566409
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
					-1.5095295634597505,
					132.37456350150978
				]
			]
		},
		{
			"type": "text",
			"version": 455,
			"versionNonce": 215499211,
			"isDeleted": false,
			"id": "bobj21M9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -467.02925952863154,
			"y": 2320.9768596222852,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 33.759979248046875,
			"height": 20,
			"seed": 1880917827,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "AYCf8n2KnLW4XAKa3FQ-R",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 3832,
			"versionNonce": 2099106917,
			"isDeleted": false,
			"id": "PjGHh6io4PNZffjmIAfaD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -451.1175556172983,
			"y": 2557.527033882968,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.580293011191884,
			"height": 211.17319165712797,
			"seed": 749978605,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1LDn5lFf9YzfJmTwbovMv",
				"focus": -0.053260101254908467,
				"gap": 11.863486842104066
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.09155681716055944,
				"gap": 10.334958232264775
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
					-1.580293011191884,
					211.17319165712797
				]
			]
		},
		{
			"type": "rectangle",
			"version": 797,
			"versionNonce": 1269026923,
			"isDeleted": false,
			"id": "uTogZR3U2ZowzQ3qqCj0J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 26.223054624053134,
			"y": 201.04379618296207,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 229,
			"height": 87,
			"seed": 1670379395,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "KS9auWrS"
				},
				{
					"id": "vhbQTHolPNpk-tVDQvwAn",
					"type": "arrow"
				},
				{
					"id": "nPRV0tyFPsQrLngjh30PC",
					"type": "arrow"
				},
				{
					"id": "YWlIUUhZ08REye1Z-HVgl",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 837,
			"versionNonce": 227540933,
			"isDeleted": false,
			"id": "KS9auWrS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 36.723054624053134,
			"y": 215.74379618296206,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 208,
			"height": 57.599999999999994,
			"seed": 1158650605,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "从当前槽开始寻找能存放该数\n据的槽，进行新值插入，遇到\n空槽时停止",
			"rawText": "从当前槽开始寻找能存放该数据的槽，进行新值插入，遇到空槽时停止",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "uTogZR3U2ZowzQ3qqCj0J",
			"originalText": "从当前槽开始寻找能存放该数据的槽，进行新值插入，遇到空槽时停止",
			"lineHeight": 1.2,
			"baseline": 52
		},
		{
			"type": "arrow",
			"version": 2007,
			"versionNonce": 1397000971,
			"isDeleted": false,
			"id": "YWlIUUhZ08REye1Z-HVgl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -322.1723949674213,
			"y": 144.45653260976115,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 450.9830132996912,
			"height": 44.350234609066376,
			"seed": 1675106573,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "xpwI8oVY"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0oide05HgsqhqlWv_nDj4",
				"focus": 0.0015553480882008097,
				"gap": 4.609077016543139
			},
			"endBinding": {
				"elementId": "uTogZR3U2ZowzQ3qqCj0J",
				"focus": 0.7765244377403524,
				"gap": 12.237028964134538
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
					273.1607327175235,
					5.622934830913465
				],
				[
					450.9830132996912,
					44.350234609066376
				]
			]
		},
		{
			"type": "text",
			"version": 265,
			"versionNonce": 1499840293,
			"isDeleted": false,
			"id": "xpwI8oVY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -48.03944720263462,
			"y": 32.555425811700985,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 1667887683,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "YWlIUUhZ08REye1Z-HVgl",
			"originalText": "true",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "diamond",
			"version": 1271,
			"versionNonce": 674558379,
			"isDeleted": false,
			"id": "1qyd9Up9kbbvRiMAmlteP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12.099020367165735,
			"y": 682.2711625027163,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 1624589731,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "aLA9nlcM"
				},
				{
					"id": "vhbQTHolPNpk-tVDQvwAn",
					"type": "arrow"
				},
				{
					"id": "lmYszLTlsjoHrEsS2Pgkz",
					"type": "arrow"
				},
				{
					"id": "2Jzxq1AsuCcE9fijlQZne",
					"type": "arrow"
				},
				{
					"id": "s_SRIRgRtSwtzD6gfnbRi",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1351,
			"versionNonce": 1460441733,
			"isDeleted": false,
			"id": "aLA9nlcM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 75.07502592136495,
			"y": 746.8711625027163,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 104.04798889160156,
			"height": 76.8,
			"seed": 1845664067,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "当前槽的\n数据key是否与\n给定\nkey相同",
			"rawText": "当前槽的\n数据key是否与给定\nkey相同",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "1qyd9Up9kbbvRiMAmlteP",
			"originalText": "当前槽的\n数据key是否与给定\nkey相同",
			"lineHeight": 1.2,
			"baseline": 71
		},
		{
			"type": "arrow",
			"version": 2320,
			"versionNonce": 1537180747,
			"isDeleted": false,
			"id": "vhbQTHolPNpk-tVDQvwAn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 137.60741847535348,
			"y": 302.9492267953359,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.6731155081983502,
			"height": 65.98903793809473,
			"seed": 167171661,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "uTogZR3U2ZowzQ3qqCj0J",
				"focus": 0.021922699375510217,
				"gap": 14.905430612373834
			},
			"endBinding": {
				"elementId": "AQKgDex8OdsNgQmhIi8AO",
				"focus": 0.027981644197215893,
				"gap": 3.8659395202354006
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
					-0.6731155081983502,
					65.98903793809473
				]
			]
		},
		{
			"type": "rectangle",
			"version": 896,
			"versionNonce": 1396832741,
			"isDeleted": false,
			"id": "ssI7SygbNLwooDH7PriRx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 18.483998134237595,
			"y": 1066.4062372504025,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 1543739853,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "EMab5nm1"
				},
				{
					"id": "-ghrUwKzysCBvFwQM59lR",
					"type": "arrow"
				},
				{
					"id": "lmYszLTlsjoHrEsS2Pgkz",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 844,
			"versionNonce": 1969144555,
			"isDeleted": false,
			"id": "EMab5nm1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 26.440007045370407,
			"y": 1109.8062372504025,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.08798217773438,
			"height": 19.2,
			"seed": 1634261037,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "更新当前槽的数据为value",
			"rawText": "更新当前槽的数据为value",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ssI7SygbNLwooDH7PriRx",
			"originalText": "更新当前槽的数据为value",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2646,
			"versionNonce": 655586629,
			"isDeleted": false,
			"id": "-ghrUwKzysCBvFwQM59lR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 113.24622170775899,
			"y": 1183.2968622504031,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 521.0614271679508,
			"height": 1598.0544311819035,
			"seed": 950299597,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ssI7SygbNLwooDH7PriRx",
				"focus": -0.03798316345285438,
				"gap": 10.890625000000682
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.34072001221652215,
				"gap": 11.464264438211359
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
					-81.34664164432502,
					526.7905590822614
				],
				[
					-521.0614271679508,
					1598.0544311819035
				]
			]
		},
		{
			"type": "arrow",
			"version": 2248,
			"versionNonce": 503131531,
			"isDeleted": false,
			"id": "lmYszLTlsjoHrEsS2Pgkz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 122.25112038492279,
			"y": 894.6312454022523,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.16235485170795982,
			"height": 167.84921059815042,
			"seed": 446509091,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "VJL8qAyN"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1qyd9Up9kbbvRiMAmlteP",
				"focus": 0.04123582331400393,
				"gap": 7.972031178957039
			},
			"endBinding": {
				"elementId": "ssI7SygbNLwooDH7PriRx",
				"focus": 0.030329852312558116,
				"gap": 3.9257812499997726
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
					-0.16235485170795982,
					167.84921059815042
				]
			]
		},
		{
			"type": "text",
			"version": 430,
			"versionNonce": 769557669,
			"isDeleted": false,
			"id": "VJL8qAyN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 115.06830480174568,
			"y": 784.62948876005,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 898436387,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "lmYszLTlsjoHrEsS2Pgkz",
			"originalText": "true",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "diamond",
			"version": 1314,
			"versionNonce": 146097195,
			"isDeleted": false,
			"id": "2M26vrb-1aSBNSmMVPVH8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 436.7975284890189,
			"y": 824.7957766620599,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 230,
			"height": 206,
			"seed": 695652973,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "XjTRT9XX"
				},
				{
					"id": "2Jzxq1AsuCcE9fijlQZne",
					"type": "arrow"
				},
				{
					"id": "nPRV0tyFPsQrLngjh30PC",
					"type": "arrow"
				},
				{
					"id": "04KhpxuYYeLkMiMCbbsoe",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1394,
			"versionNonce": 2068883461,
			"isDeleted": false,
			"id": "XjTRT9XX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 507.7735340432181,
			"y": 898.9957766620599,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 88.04798889160156,
			"height": 57.599999999999994,
			"seed": 1496054989,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "当前槽的\n数据key是否\n为null",
			"rawText": "当前槽的\n数据key是否\n为null",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "2M26vrb-1aSBNSmMVPVH8",
			"originalText": "当前槽的\n数据key是否\n为null",
			"lineHeight": 1.2,
			"baseline": 52
		},
		{
			"type": "arrow",
			"version": 2236,
			"versionNonce": 493979339,
			"isDeleted": false,
			"id": "2Jzxq1AsuCcE9fijlQZne",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 216.43183049615894,
			"y": 740.5543515144416,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 327.72600840396376,
			"height": 78.9046088022867,
			"seed": 1030416771,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Ft2vYUcu"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1qyd9Up9kbbvRiMAmlteP",
				"focus": -0.4908691466198909,
				"gap": 16.185192081684505
			},
			"endBinding": {
				"elementId": "2M26vrb-1aSBNSmMVPVH8",
				"focus": 1.0158835514475328,
				"gap": 9.072405379636635
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
					190.57090192407267,
					12.464073669996878
				],
				[
					327.72600840396376,
					78.9046088022867
				]
			]
		},
		{
			"type": "text",
			"version": 432,
			"versionNonce": 1365729125,
			"isDeleted": false,
			"id": "Ft2vYUcu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 419.97995153277327,
			"y": 666.2496249236881,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1064587245,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "2Jzxq1AsuCcE9fijlQZne",
			"originalText": "false",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2198,
			"versionNonce": 1657566571,
			"isDeleted": false,
			"id": "nPRV0tyFPsQrLngjh30PC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 620.0618016835781,
			"y": 871.1294439056419,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 349.8452278833886,
			"height": 600.6050788990146,
			"seed": 257585517,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "5Fhf6okP"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2M26vrb-1aSBNSmMVPVH8",
				"focus": 0.6566075381011032,
				"gap": 11.030050978258828
			},
			"endBinding": {
				"elementId": "uTogZR3U2ZowzQ3qqCj0J",
				"focus": -0.5957085791178863,
				"gap": 14.993519176136374
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
					-43.643091669634146,
					-341.3238530728627
				],
				[
					-349.8452278833886,
					-600.6050788990146
				]
			]
		},
		{
			"type": "text",
			"version": 443,
			"versionNonce": 1450191557,
			"isDeleted": false,
			"id": "5Fhf6okP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 462.83472563894395,
			"y": 510.60559083277917,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 227.16796875,
			"height": 38.4,
			"seed": 293233251,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "false\n使用开放地址放重新计算hash槽",
			"rawText": "false\n使用开放地址放重新计算hash槽",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "nPRV0tyFPsQrLngjh30PC",
			"originalText": "false\n使用开放地址放重新计算hash槽",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "rectangle",
			"version": 918,
			"versionNonce": 672174091,
			"isDeleted": false,
			"id": "VpvvT94PCb9K2NfVd2B10",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 452.83632365030746,
			"y": 1143.268659014599,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 201,
			"height": 106,
			"seed": 2041756077,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "DebJ7p0F"
				},
				{
					"id": "04KhpxuYYeLkMiMCbbsoe",
					"type": "arrow"
				},
				{
					"id": "Pq6dqDtDC3xGQ9RJLzCDM",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 867,
			"versionNonce": 185109029,
			"isDeleted": false,
			"id": "DebJ7p0F",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 460.43236246866684,
			"y": 1167.468659014599,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.80792236328125,
			"height": 57.599999999999994,
			"seed": 1609104397,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "认定当前槽是一个脏数据\n，记为staleSlot，从stal\neSlot开始清理并更新数据",
			"rawText": "认定当前槽是一个脏数据，记为staleSlot，从staleSlot开始清理并更新数据",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "VpvvT94PCb9K2NfVd2B10",
			"originalText": "认定当前槽是一个脏数据，记为staleSlot，从staleSlot开始清理并更新数据",
			"lineHeight": 1.2,
			"baseline": 52
		},
		{
			"type": "rectangle",
			"version": 976,
			"versionNonce": 1865963179,
			"isDeleted": false,
			"id": "CoQ_g_7z03BwYj7CzfHvD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 452.83632365030746,
			"y": 1363.3254771964175,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 201,
			"height": 106,
			"seed": 1891669325,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AyW2Y6C4"
				},
				{
					"id": "Pq6dqDtDC3xGQ9RJLzCDM",
					"type": "arrow"
				},
				{
					"id": "oMfxFNtmZ9ABrTviZrzJM",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 935,
			"versionNonce": 1965445,
			"isDeleted": false,
			"id": "AyW2Y6C4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 464.57635966104965,
			"y": 1397.1254771964175,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 177.51992797851562,
			"height": 38.4,
			"seed": 1543136173,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "从当前槽开始往前查找脏\n槽，记为slotToExpunge",
			"rawText": "从当前槽开始往前查找脏槽，记为slotToExpunge",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "CoQ_g_7z03BwYj7CzfHvD",
			"originalText": "从当前槽开始往前查找脏槽，记为slotToExpunge",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "rectangle",
			"version": 1026,
			"versionNonce": 384715083,
			"isDeleted": false,
			"id": "NSwFDKRVBEpY2lpoJzuRM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 452.83632365030746,
			"y": 1565.89649992369,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 201,
			"height": 106,
			"seed": 48093645,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "upHm9JqX"
				},
				{
					"id": "oMfxFNtmZ9ABrTviZrzJM",
					"type": "arrow"
				},
				{
					"id": "kqR6RUHJxSRRMLm9BAPvx",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 987,
			"versionNonce": 1290667237,
			"isDeleted": false,
			"id": "upHm9JqX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 460.73634806436996,
			"y": 1599.69649992369,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.199951171875,
			"height": 38.4,
			"seed": 855316525,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "从staleSlot开始往后查找\n脏槽",
			"rawText": "从staleSlot开始往后查找脏槽",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "NSwFDKRVBEpY2lpoJzuRM",
			"originalText": "从staleSlot开始往后查找脏槽",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "diamond",
			"version": 1251,
			"versionNonce": 1150085099,
			"isDeleted": false,
			"id": "qLcFjKvWDXMtoAljewKg8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 449.82069865030724,
			"y": 1752.3098521964175,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 988665613,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "lB4IWJ2t"
				},
				{
					"id": "kqR6RUHJxSRRMLm9BAPvx",
					"type": "arrow"
				},
				{
					"id": "PChAFJtfnID7nKNMxa3bd",
					"type": "arrow"
				},
				{
					"id": "BB36ZpmNOq3D6C71EsqCZ",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1333,
			"versionNonce": 1158029381,
			"isDeleted": false,
			"id": "lB4IWJ2t",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 520.7967042045065,
			"y": 1836.1098521964175,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 88.04798889160156,
			"height": 38.4,
			"seed": 2056875373,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "查找到\nkey相同的槽",
			"rawText": "查找到\nkey相同的槽",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "qLcFjKvWDXMtoAljewKg8",
			"originalText": "查找到\nkey相同的槽",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "rectangle",
			"version": 1069,
			"versionNonce": 236892811,
			"isDeleted": false,
			"id": "DvebHZK2RZWWrE3NkMmhg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 656.57922137758,
			"y": 2077.579738560056,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 201,
			"height": 106,
			"seed": 44836803,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "AKE26X9p"
				},
				{
					"id": "PChAFJtfnID7nKNMxa3bd",
					"type": "arrow"
				},
				{
					"id": "HXqJt1_2265Ap2VnR2h-z",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1025,
			"versionNonce": 1849937829,
			"isDeleted": false,
			"id": "AKE26X9p",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 664.4792457916425,
			"y": 2101.7797385600556,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.199951171875,
			"height": 57.599999999999994,
			"seed": 318256995,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "将value更新到槽中，并将\n槽数据与staleSlot位置互\n换",
			"rawText": "将value更新到槽中，并将槽数据与staleSlot位置互换",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "DvebHZK2RZWWrE3NkMmhg",
			"originalText": "将value更新到槽中，并将槽数据与staleSlot位置互换",
			"lineHeight": 1.2,
			"baseline": 52
		},
		{
			"type": "rectangle",
			"version": 1174,
			"versionNonce": 1683892523,
			"isDeleted": false,
			"id": "LfZ6_qXkksKdwHzCBngwS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 656.57922137758,
			"y": 2353.450106953032,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 201,
			"height": 106,
			"seed": 155561421,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Iq4OQuY9"
				},
				{
					"id": "HXqJt1_2265Ap2VnR2h-z",
					"type": "arrow"
				},
				{
					"id": "2DgazU-DKk52e1D_qwETa",
					"type": "arrow"
				},
				{
					"id": "YQPb_DuNh1mKFEvRj_EBI",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1143,
			"versionNonce": 919900933,
			"isDeleted": false,
			"id": "Iq4OQuY9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 668.3192573883222,
			"y": 2386.450106953032,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 177.51992797851562,
			"height": 40,
			"seed": 1162374189,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "从slotToExpunge开始清\n理槽中key为null的脏数据",
			"rawText": "从slotToExpunge开始清理槽中key为null的脏数据",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "LfZ6_qXkksKdwHzCBngwS",
			"originalText": "从slotToExpunge开始清理槽中key为null的脏数据",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 1185,
			"versionNonce": 1018547147,
			"isDeleted": false,
			"id": "Bq0G5TkPGbYTHJM7woWyM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 656.57922137758,
			"y": 2561.021976565465,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 128660579,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "3U6X046w"
				},
				{
					"id": "2DgazU-DKk52e1D_qwETa",
					"type": "arrow"
				},
				{
					"id": "4XIY0Qdaz4mUGbX4I3hHt",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1174,
			"versionNonce": 41114085,
			"isDeleted": false,
			"id": "3U6X046w",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 669.07922137758,
			"y": 2585.221976565465,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 176,
			"height": 57.599999999999994,
			"seed": 162312195,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243286347,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "启发式贪心清理其它的脏\n槽，按比例进行最大努力\n清理",
			"rawText": "启发式贪心清理其它的脏槽，按比例进行最大努力清理",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Bq0G5TkPGbYTHJM7woWyM",
			"originalText": "启发式贪心清理其它的脏槽，按比例进行最大努力清理",
			"lineHeight": 1.2,
			"baseline": 52
		},
		{
			"type": "rectangle",
			"version": 1321,
			"versionNonce": 2067798635,
			"isDeleted": false,
			"id": "FU9MnYQBBpelNIyU08ezs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 128.26685451459912,
			"y": 2090.256246825354,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#99e9f2",
			"width": 201,
			"height": 106,
			"seed": 2044517411,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "OdyXTW0c"
				},
				{
					"id": "BB36ZpmNOq3D6C71EsqCZ",
					"type": "arrow"
				},
				{
					"id": "y-DKJ7x2coeUHulAPtXx6",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1275,
			"versionNonce": 1815434693,
			"isDeleted": false,
			"id": "OdyXTW0c",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 176.16687892866162,
			"y": 2124.0562468253543,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 105.199951171875,
			"height": 38.4,
			"seed": 1590910915,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "将value覆盖到\nstaleSlot槽中",
			"rawText": "将value覆盖到\nstaleSlot槽中",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "FU9MnYQBBpelNIyU08ezs",
			"originalText": "将value覆盖到\nstaleSlot槽中",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "diamond",
			"version": 1661,
			"versionNonce": 482841867,
			"isDeleted": false,
			"id": "Cn0ySZnfSN_0EYNXnVud-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 107.88493549684824,
			"y": 2317.0528185245207,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 212,
			"seed": 51778147,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "L3nS944u"
				},
				{
					"id": "gWYpx7rL6q0SJw2hdApu2",
					"type": "arrow"
				},
				{
					"id": "YQPb_DuNh1mKFEvRj_EBI",
					"type": "arrow"
				},
				{
					"id": "y-DKJ7x2coeUHulAPtXx6",
					"type": "arrow"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1737,
			"versionNonce": 619730213,
			"isDeleted": false,
			"id": "L3nS944u",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 170.50097095827402,
			"y": 2375.0528185245207,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 104.76792907714844,
			"height": 96,
			"seed": 1342950915,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "slotToExpung\ne\n是否与staleSl\not\n相等",
			"rawText": "slotToExpunge\n是否与staleSlot\n相等",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Cn0ySZnfSN_0EYNXnVud-",
			"originalText": "slotToExpunge\n是否与staleSlot\n相等",
			"lineHeight": 1.2,
			"baseline": 90
		},
		{
			"type": "arrow",
			"version": 1819,
			"versionNonce": 416644165,
			"isDeleted": false,
			"id": "04KhpxuYYeLkMiMCbbsoe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 551.2465354502342,
			"y": 1033.2501889907037,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.7222395598114417,
			"height": 92.78409502389445,
			"seed": 2075435555,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Zqon31a5"
				}
			],
			"updated": 1693243151395,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2M26vrb-1aSBNSmMVPVH8",
				"gap": 2.5154986002358157,
				"focus": -0.012229829408036675
			},
			"endBinding": {
				"elementId": "VpvvT94PCb9K2NfVd2B10",
				"gap": 17.23437500000091,
				"focus": -0.050409082395334155
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
					-1.7222395598114417,
					92.78409502389445
				]
			]
		},
		{
			"type": "text",
			"version": 430,
			"versionNonce": 1087662213,
			"isDeleted": false,
			"id": "Zqon31a5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 532.7072095669239,
			"y": 1062.7687785451594,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 450371139,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "04KhpxuYYeLkMiMCbbsoe",
			"originalText": "true",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1338,
			"versionNonce": 1803349765,
			"isDeleted": false,
			"id": "Pq6dqDtDC3xGQ9RJLzCDM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 548.790869104853,
			"y": 1259.670647650962,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.9232954545453822,
			"height": 89.40340909090901,
			"seed": 707367181,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243151395,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "VpvvT94PCb9K2NfVd2B10",
				"gap": 10.401988636363058,
				"focus": 0.03850356867339114
			},
			"endBinding": {
				"elementId": "CoQ_g_7z03BwYj7CzfHvD",
				"gap": 14.251420454546405,
				"focus": -0.060993935190292455
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
					-0.9232954545453822,
					89.40340909090901
				]
			]
		},
		{
			"type": "arrow",
			"version": 1403,
			"versionNonce": 564072421,
			"isDeleted": false,
			"id": "oMfxFNtmZ9ABrTviZrzJM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 555.0709129551843,
			"y": 1480.0003725990962,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.1404106951230233,
			"height": 74.59215005186593,
			"seed": 1811474573,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CoQ_g_7z03BwYj7CzfHvD",
				"focus": -0.016051021619844858,
				"gap": 10.674895402678658
			},
			"endBinding": {
				"elementId": "NSwFDKRVBEpY2lpoJzuRM",
				"focus": 0.019841442307800612,
				"gap": 11.303977272727934
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
					0.1404106951230233,
					74.59215005186593
				]
			]
		},
		{
			"type": "arrow",
			"version": 1371,
			"versionNonce": 394661099,
			"isDeleted": false,
			"id": "kqR6RUHJxSRRMLm9BAPvx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 554.5153009230347,
			"y": 1686.1905340145997,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.06392045454549589,
			"height": 64.98579545454413,
			"seed": 564617187,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "NSwFDKRVBEpY2lpoJzuRM",
				"focus": -0.011066761159842883,
				"gap": 14.29403409090969
			},
			"endBinding": {
				"elementId": "qLcFjKvWDXMtoAljewKg8",
				"focus": -0.08816565962547511,
				"gap": 7.677205553195336
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
					0.06392045454549589,
					64.98579545454413
				]
			]
		},
		{
			"type": "arrow",
			"version": 1447,
			"versionNonce": 358468421,
			"isDeleted": false,
			"id": "PChAFJtfnID7nKNMxa3bd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 583.6630281957621,
			"y": 1967.284284014599,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 178.77840909090855,
			"height": 97.07386363636306,
			"seed": 2071562851,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "vEUtpMHM"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qLcFjKvWDXMtoAljewKg8",
				"focus": 0.9877994494962,
				"gap": 19.256177852453305
			},
			"endBinding": {
				"elementId": "DvebHZK2RZWWrE3NkMmhg",
				"focus": 0.6426814341830526,
				"gap": 13.22159090909372
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
					178.77840909090855,
					97.07386363636306
				]
			]
		},
		{
			"type": "text",
			"version": 430,
			"versionNonce": 1353398155,
			"isDeleted": false,
			"id": "vEUtpMHM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 656.1722431171931,
			"y": 2006.2212158327807,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 1388191341,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "PChAFJtfnID7nKNMxa3bd",
			"originalText": "true",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1869,
			"versionNonce": 1282284197,
			"isDeleted": false,
			"id": "BB36ZpmNOq3D6C71EsqCZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 549.8709810306563,
			"y": 1967.982253960295,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 351.76708728289555,
			"height": 110.24467116062533,
			"seed": 1735162093,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "21n3w6UH"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qLcFjKvWDXMtoAljewKg8",
				"focus": -1.0484186743452553,
				"gap": 17.179050000101313
			},
			"endBinding": {
				"elementId": "FU9MnYQBBpelNIyU08ezs",
				"focus": -0.8833356869992941,
				"gap": 12.029321704433642
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
					-351.76708728289555,
					110.24467116062533
				]
			]
		},
		{
			"type": "text",
			"version": 431,
			"versionNonce": 219287083,
			"isDeleted": false,
			"id": "21n3w6UH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 384.8188821608277,
			"y": 1985.0848521964167,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1173447875,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "BB36ZpmNOq3D6C71EsqCZ",
			"originalText": "false",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1546,
			"versionNonce": 259882501,
			"isDeleted": false,
			"id": "HXqJt1_2265Ap2VnR2h-z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 751.8410877956463,
			"y": 2195.3169544691486,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 2.0481094914279083,
			"height": 151.55218657479236,
			"seed": 956240781,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "DvebHZK2RZWWrE3NkMmhg",
				"focus": 0.06039550268359687,
				"gap": 11.73721590909281
			},
			"endBinding": {
				"elementId": "LfZ6_qXkksKdwHzCBngwS",
				"focus": -0.02356176763186442,
				"gap": 6.580965909090992
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
					2.0481094914279083,
					151.55218657479236
				]
			]
		},
		{
			"type": "arrow",
			"version": 1590,
			"versionNonce": 58591237,
			"isDeleted": false,
			"id": "2DgazU-DKk52e1D_qwETa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 759.0197202674158,
			"y": 2469.660334225759,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.7833980776404132,
			"height": 82.39431279425071,
			"seed": 1315720163,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243288251,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "LfZ6_qXkksKdwHzCBngwS",
				"gap": 10.210227272727025,
				"focus": -0.013261876007534427
			},
			"endBinding": {
				"elementId": "Bq0G5TkPGbYTHJM7woWyM",
				"gap": 8.967329545455414,
				"focus": 0.0328014700154386
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
					0.7833980776404132,
					82.39431279425071
				]
			]
		},
		{
			"type": "arrow",
			"version": 3127,
			"versionNonce": 753450853,
			"isDeleted": false,
			"id": "4XIY0Qdaz4mUGbX4I3hHt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 759.3471226369676,
			"y": 2684.6433332388583,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1130.9363098398053,
			"height": 175.82184661889278,
			"seed": 250464493,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693243288251,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Bq0G5TkPGbYTHJM7woWyM",
				"gap": 17.621356673393166,
				"focus": -0.5490380824285898
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.37994641289785386,
				"gap": 13.73171904771246
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
					-160.07374929896855,
					125.62054157320517
				],
				[
					-1130.9363098398053,
					175.82184661889278
				]
			]
		},
		{
			"type": "arrow",
			"version": 2776,
			"versionNonce": 1509867371,
			"isDeleted": false,
			"id": "gWYpx7rL6q0SJw2hdApu2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 136.3148003454341,
			"y": 2481.576568232668,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 523.9996968447017,
			"height": 331.4400871278667,
			"seed": 427700675,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "IraALWd2"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Cn0ySZnfSN_0EYNXnVud-",
				"focus": -0.03553246272945654,
				"gap": 23.763826749587807
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.2227757853619715,
				"gap": 2.6565966958547733
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
					-523.9996968447017,
					331.4400871278667
				]
			]
		},
		{
			"type": "text",
			"version": 362,
			"versionNonce": 1507819717,
			"isDeleted": false,
			"id": "IraALWd2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -77.11451350572312,
			"y": 2401.140763001072,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 2053958701,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "gWYpx7rL6q0SJw2hdApu2",
			"originalText": "true",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2258,
			"versionNonce": 1449761291,
			"isDeleted": false,
			"id": "YQPb_DuNh1mKFEvRj_EBI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 344.69742555767,
			"y": 2417.0866822629955,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 300.3776422586279,
			"height": 1.814610214086315,
			"seed": 2063412589,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "GZfGxkwO"
				}
			],
			"updated": 1693242755477,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Cn0ySZnfSN_0EYNXnVud-",
				"focus": -0.04934203111757483,
				"gap": 9.004018319143114
			},
			"endBinding": {
				"elementId": "LfZ6_qXkksKdwHzCBngwS",
				"focus": -0.15194503440076848,
				"gap": 11.504153561282124
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
					300.3776422586279,
					-1.814610214086315
				]
			]
		},
		{
			"type": "text",
			"version": 429,
			"versionNonce": 1377640485,
			"isDeleted": false,
			"id": "GZfGxkwO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 480.1636061868895,
			"y": 2336.268388853379,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1939684429,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "YQPb_DuNh1mKFEvRj_EBI",
			"originalText": "false",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2083,
			"versionNonce": 1105207467,
			"isDeleted": false,
			"id": "y-DKJ7x2coeUHulAPtXx6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 223.1584324356005,
			"y": 2203.3810439032763,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 3.298079770879781,
			"height": 109.65219666431312,
			"seed": 120191533,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "FU9MnYQBBpelNIyU08ezs",
				"focus": 0.07264703092690769,
				"gap": 7.124797077922267
			},
			"endBinding": {
				"elementId": "Cn0ySZnfSN_0EYNXnVud-",
				"focus": 0.059832229950985825,
				"gap": 5.376201462753059
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
					3.298079770879781,
					109.65219666431312
				]
			]
		},
		{
			"type": "arrow",
			"version": 2860,
			"versionNonce": 1680855941,
			"isDeleted": false,
			"id": "xJj1W6f_5J-goSJcBq56-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -547.102903533244,
			"y": 2184.5135094509574,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 245.6872809179103,
			"height": 652.1704340864371,
			"seed": 63986093,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "RID7hBEU"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "T3l9HpdHo4Bku8vxH8bYJ",
				"focus": 0.6842439638077371,
				"gap": 10.507553313788193
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": -0.980042449831174,
				"gap": 7.920522577473989
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
					-244.70655786991233,
					196.49351990125274
				],
				[
					0.9807230479979694,
					652.1704340864371
				]
			]
		},
		{
			"type": "text",
			"version": 549,
			"versionNonce": 369939275,
			"isDeleted": false,
			"id": "RID7hBEU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -737.8817137926278,
			"y": 1693.1177453999148,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1337919139,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "xJj1W6f_5J-goSJcBq56-",
			"originalText": "false",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "text",
			"version": 528,
			"versionNonce": 1012210405,
			"isDeleted": false,
			"id": "C2CbM31X",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 898.5530043619804,
			"y": 2590.595304057583,
			"strokeColor": "#1971c2",
			"backgroundColor": "#fcc2d7",
			"width": 176,
			"height": 60,
			"seed": 1940926893,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0",
			"rawText": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0",
			"lineHeight": 1.25,
			"baseline": 54
		},
		{
			"type": "text",
			"version": 786,
			"versionNonce": 860500459,
			"isDeleted": false,
			"id": "1bezDJzh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 302.60560243095415,
			"y": 230.11731104191773,
			"strokeColor": "#1971c2",
			"backgroundColor": "#d0bfff",
			"width": 144,
			"height": 20,
			"seed": 39648227,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "使用开放地址法寻址",
			"rawText": "使用开放地址法寻址",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "使用开放地址法寻址",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "diamond",
			"version": 968,
			"versionNonce": 588936773,
			"isDeleted": false,
			"id": "iLXE5W5Lx1BfsXxEUTs4c",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -519.6082134994396,
			"y": 689.269934409193,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 181,
			"height": 183,
			"seed": 1986888067,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "PEtP0Pni"
				},
				{
					"id": "XvLzY_87WT1etuKK5faAW",
					"type": "arrow"
				},
				{
					"id": "jpMeRYHlOZfoOBOvnr2w4",
					"type": "arrow"
				},
				{
					"id": "1vXz6kIEeCJWzvTstvh13",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 913,
			"versionNonce": 178437259,
			"isDeleted": false,
			"id": "PEtP0Pni",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -468.85821349943956,
			"y": 761.8199344091929,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 80,
			"height": 38.4,
			"seed": 1088968995,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "找到了过期\n的数据？",
			"rawText": "找到了过期的数据？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "iLXE5W5Lx1BfsXxEUTs4c",
			"originalText": "找到了过期的数据？",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "text",
			"version": 699,
			"versionNonce": 1935500709,
			"isDeleted": false,
			"id": "t8JA2zsR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -347.30541439820604,
			"y": 714.6821147045496,
			"strokeColor": "#1971c2",
			"backgroundColor": "#d0bfff",
			"width": 184.04798889160156,
			"height": 40,
			"seed": 794496013,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "过期数据：key已经被回收\n但value未被回收",
			"rawText": "过期数据：key已经被回收\n但value未被回收",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "过期数据：key已经被回收\n但value未被回收",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "rectangle",
			"version": 1174,
			"versionNonce": 131134251,
			"isDeleted": false,
			"id": "UazpQQDxoCA1Irisu8SLu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -530.9982647155019,
			"y": 943.2992652573614,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 1398348707,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "xZppoh6a"
				},
				{
					"id": "jpMeRYHlOZfoOBOvnr2w4",
					"type": "arrow"
				},
				{
					"id": "P1nU3PADse2jEjjXpEaDu",
					"type": "arrow"
				},
				{
					"id": "Gs6pFPMYMJ0IKqH0wKXUd",
					"type": "arrow"
				},
				{
					"id": "TG5lRPxWyBKybCqK_JuLq",
					"type": "arrow"
				},
				{
					"id": "ImqDWpKayT1X0uRagq8S7",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1261,
			"versionNonce": 1347119365,
			"isDeleted": false,
			"id": "xZppoh6a",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -494.49826471550193,
			"y": 986.6992652573614,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 128,
			"height": 19.2,
			"seed": 1302048579,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "清理当前过期数据",
			"rawText": "清理当前过期数据",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "UazpQQDxoCA1Irisu8SLu",
			"originalText": "清理当前过期数据",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"type": "diamond",
			"version": 1079,
			"versionNonce": 994006475,
			"isDeleted": false,
			"id": "MOqctMbP1r4vY5TYSkUPy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -530.0776929001744,
			"y": 1314.05806029938,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 181,
			"height": 183,
			"seed": 1314723117,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "VnPM0VWB"
				},
				{
					"id": "wyBVBCs-6Gjk6ejaoSxWT",
					"type": "arrow"
				},
				{
					"id": "ImqDWpKayT1X0uRagq8S7",
					"type": "arrow"
				},
				{
					"id": "J14jObGbx7PC25pLNKDCx",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1071,
			"versionNonce": 459407461,
			"isDeleted": false,
			"id": "VnPM0VWB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -479.3276929001744,
			"y": 1377.00806029938,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 80,
			"height": 57.599999999999994,
			"seed": 1975248781,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "找到了其他\n过期的数据\n？",
			"rawText": "找到了其他过期的数据？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "MOqctMbP1r4vY5TYSkUPy",
			"originalText": "找到了其他过期的数据？",
			"lineHeight": 1.2,
			"baseline": 52
		},
		{
			"type": "diamond",
			"version": 1525,
			"versionNonce": 1141400683,
			"isDeleted": false,
			"id": "ed0-Q7RYXL6OhosPGyes7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -529.79587598326,
			"y": 1581.7592090427092,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 176,
			"height": 242,
			"seed": 19986979,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "9eXw2dWN"
				},
				{
					"id": "wyBVBCs-6Gjk6ejaoSxWT",
					"type": "arrow"
				},
				{
					"id": "eGKZYb_A7u4UWOToPz_nz",
					"type": "arrow"
				},
				{
					"id": "Gs6pFPMYMJ0IKqH0wKXUd",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1606,
			"versionNonce": 711896005,
			"isDeleted": false,
			"id": "9eXw2dWN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -480.29586072447097,
			"y": 1656.536986820487,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 76.99996948242188,
			"height": 92.4444444444444,
			"seed": 1643363779,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 15.407407407407401,
			"fontFamily": 4,
			"text": "计算未过期\n数据的哈希\n槽，是否应\n位于当前位\n置？",
			"rawText": "计算未过期数据的哈希槽，是否应位于当前位置？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ed0-Q7RYXL6OhosPGyes7",
			"originalText": "计算未过期数据的哈希槽，是否应位于当前位置？",
			"lineHeight": 1.2,
			"baseline": 86
		},
		{
			"type": "rectangle",
			"version": 1340,
			"versionNonce": 2046205707,
			"isDeleted": false,
			"id": "uyYqxenN6jX5fD1o9M88_",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -543.8026941951804,
			"y": 1904.0640299349293,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 1883649325,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "86i4X2V2"
				},
				{
					"id": "eGKZYb_A7u4UWOToPz_nz",
					"type": "arrow"
				},
				{
					"id": "TG5lRPxWyBKybCqK_JuLq",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1482,
			"versionNonce": 1032776485,
			"isDeleted": false,
			"id": "86i4X2V2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -532.7026728328757,
			"y": 1937.8640299349292,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 178.79995727539062,
			"height": 38.4,
			"seed": 1031866253,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "对该元素重新rehash放到\n合适的位置",
			"rawText": "对该元素重新rehash放到合适的位置",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "uyYqxenN6jX5fD1o9M88_",
			"originalText": "对该元素重新rehash放到合适的位置",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "arrow",
			"version": 1685,
			"versionNonce": 1468302763,
			"isDeleted": false,
			"id": "XvLzY_87WT1etuKK5faAW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -433.84150411697465,
			"y": 644.3349377100119,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 0.47144300109005144,
			"height": 40.37593913660953,
			"seed": 1798138403,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "EtmeMz-BdnDUhKY0TuO1E",
				"focus": -0.07041612829006881,
				"gap": 6.555443033375013
			},
			"endBinding": {
				"elementId": "iLXE5W5Lx1BfsXxEUTs4c",
				"focus": -0.06990443511386588,
				"gap": 6.906447285657563
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
					-0.47144300109005144,
					40.37593913660953
				]
			]
		},
		{
			"type": "arrow",
			"version": 1772,
			"versionNonce": 892354181,
			"isDeleted": false,
			"id": "jpMeRYHlOZfoOBOvnr2w4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -429.00270624946984,
			"y": 873.264352942912,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 0.4496120473654628,
			"height": 69.03491231444934,
			"seed": 1549583245,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "JW58vCOM"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "iLXE5W5Lx1BfsXxEUTs4c",
				"focus": 0.005490523416444006,
				"gap": 1
			},
			"endBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"focus": 0.022776129452370553,
				"gap": 1.0000000000001137
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
					0.4496120473654628,
					69.03491231444934
				]
			]
		},
		{
			"type": "text",
			"version": 442,
			"versionNonce": 601659467,
			"isDeleted": false,
			"id": "JW58vCOM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -327.92786995038557,
			"y": 903.7549286676581,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 33.759979248046875,
			"height": 20,
			"seed": 541429933,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "jpMeRYHlOZfoOBOvnr2w4",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 1964,
			"versionNonce": 135276005,
			"isDeleted": false,
			"id": "P1nU3PADse2jEjjXpEaDu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -432.1442862347225,
			"y": 1055.9844177703853,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 1.7906001385542822,
			"height": 74.49885000974632,
			"seed": 1068048483,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"focus": 0.030268775507038204,
				"gap": 6.685152513023922
			},
			"endBinding": {
				"elementId": "rB1uWGMBnujG2mAKY5_0f",
				"focus": 0.06557467280064828,
				"gap": 4.613668440299307
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
					1.7906001385542822,
					74.49885000974632
				]
			]
		},
		{
			"type": "arrow",
			"version": 2670,
			"versionNonce": 1557747435,
			"isDeleted": false,
			"id": "wyBVBCs-6Gjk6ejaoSxWT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -438.2481837979967,
			"y": 1500.8402819235723,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 2.552097740356203,
			"height": 80.41751902625788,
			"seed": 77034403,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "zIPMfYVZ"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "MOqctMbP1r4vY5TYSkUPy",
				"focus": -0.04810328188100845,
				"gap": 3.6049557607296236
			},
			"endBinding": {
				"elementId": "ed0-Q7RYXL6OhosPGyes7",
				"focus": -0.03250369229431971,
				"gap": 1.100087202144806
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
					-2.552097740356203,
					80.41751902625788
				]
			]
		},
		{
			"type": "text",
			"version": 454,
			"versionNonce": 2033983813,
			"isDeleted": false,
			"id": "zIPMfYVZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -459.5946660354002,
			"y": 1531.756515352744,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 40.09596252441406,
			"height": 20,
			"seed": 35126115,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "wyBVBCs-6Gjk6ejaoSxWT",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2488,
			"versionNonce": 1627734411,
			"isDeleted": false,
			"id": "eGKZYb_A7u4UWOToPz_nz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -444.1489777851736,
			"y": 1826.0148666855116,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 0.7139301968582572,
			"height": 71.77845665326072,
			"seed": 2029831021,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "uZNRrXoC"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ed0-Q7RYXL6OhosPGyes7",
				"focus": 0.04067090657589459,
				"gap": 3.2297523223871565
			},
			"endBinding": {
				"elementId": "uyYqxenN6jX5fD1o9M88_",
				"focus": 0.004525228396347675,
				"gap": 6.27070659615697
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
					0.7139301968582572,
					71.77845665326072
				]
			]
		},
		{
			"type": "text",
			"version": 457,
			"versionNonce": 236040357,
			"isDeleted": false,
			"id": "uZNRrXoC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -336.39826379104306,
			"y": 1658.1937491251,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 40.09596252441406,
			"height": 20,
			"seed": 1742141091,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "eGKZYb_A7u4UWOToPz_nz",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2491,
			"versionNonce": 1503320107,
			"isDeleted": false,
			"id": "Gs6pFPMYMJ0IKqH0wKXUd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -370.153939007786,
			"y": 1654.764642253196,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 104.84160987026758,
			"height": 485.01543201850814,
			"seed": 1310277389,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "ZMts38zj"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ed0-Q7RYXL6OhosPGyes7",
				"focus": 0.6578546310042251,
				"gap": 14.999689990526491
			},
			"endBinding": {
				"elementId": "rB1uWGMBnujG2mAKY5_0f",
				"focus": -1.0040253946626143,
				"gap": 16.127739631003465
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
					104.84160987026758,
					-365.9316886857275
				],
				[
					51.1389645587069,
					-485.01543201850814
				]
			]
		},
		{
			"type": "text",
			"version": 449,
			"versionNonce": 1144935429,
			"isDeleted": false,
			"id": "ZMts38zj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -265.67210323203653,
			"y": 1233.6485902568975,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 33.759979248046875,
			"height": 20,
			"seed": 984133261,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Gs6pFPMYMJ0IKqH0wKXUd",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2286,
			"versionNonce": 1686411979,
			"isDeleted": false,
			"id": "ImqDWpKayT1X0uRagq8S7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -526.9887018519747,
			"y": 1391.1058111543634,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 117.8718678585468,
			"height": 420.66227136220687,
			"seed": 1602269261,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "o164zHVU"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "MOqctMbP1r4vY5TYSkUPy",
				"focus": -0.8894413273924682,
				"gap": 7.9667653377101
			},
			"endBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"focus": 0.9308852189414175,
				"gap": 7.391796936974629
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
					-117.8718678585468,
					-246.2946072146517
				],
				[
					-11.401359800501837,
					-420.66227136220687
				]
			]
		},
		{
			"type": "text",
			"version": 455,
			"versionNonce": 129993573,
			"isDeleted": false,
			"id": "o164zHVU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -703.0258674740462,
			"y": 934.619090000113,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 33.759979248046875,
			"height": 20,
			"seed": 204949357,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "true",
			"rawText": "true",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ImqDWpKayT1X0uRagq8S7",
			"originalText": "true",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 2753,
			"versionNonce": 2120026475,
			"isDeleted": false,
			"id": "TG5lRPxWyBKybCqK_JuLq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -371.578609797988,
			"y": 1900.3291171907067,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 217.99383395002627,
			"height": 892.8392887822636,
			"seed": 1836936269,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "Pq9K9J1q"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "uyYqxenN6jX5fD1o9M88_",
				"focus": 0.4470569620876232,
				"gap": 3.7349127442225836
			},
			"endBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"focus": -0.8748987681174515,
				"gap": 24.170505760866945
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
					217.99383395002627,
					-654.3423395702789
				],
				[
					65.75085084335302,
					-892.8392887822636
				]
			]
		},
		{
			"type": "text",
			"version": 564,
			"versionNonce": 1759696581,
			"isDeleted": false,
			"id": "Pq9K9J1q",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -46.65915828980383,
			"y": 1236.755295362239,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 96,
			"height": 20,
			"seed": 897973869,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "继续循环寻找",
			"rawText": "继续循环寻找",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "TG5lRPxWyBKybCqK_JuLq",
			"originalText": "继续循环寻找",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "rectangle",
			"version": 1251,
			"versionNonce": 745448459,
			"isDeleted": false,
			"id": "rB1uWGMBnujG2mAKY5_0f",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -536.1427140800826,
			"y": 1135.096936220431,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 201,
			"height": 106,
			"seed": 1545418051,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"type": "text",
					"id": "GAmFZmDr"
				},
				{
					"id": "P1nU3PADse2jEjjXpEaDu",
					"type": "arrow"
				},
				{
					"id": "J14jObGbx7PC25pLNKDCx",
					"type": "arrow"
				},
				{
					"id": "Gs6pFPMYMJ0IKqH0wKXUd",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1358,
			"versionNonce": 306558501,
			"isDeleted": false,
			"id": "GAmFZmDr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -523.6427140800826,
			"y": 1168.896936220431,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 176,
			"height": 38.4,
			"seed": 99325155,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "往后寻找其他过期数据，\n直到遇到空槽",
			"rawText": "往后寻找其他过期数据，直到遇到空槽",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "rB1uWGMBnujG2mAKY5_0f",
			"originalText": "往后寻找其他过期数据，直到遇到空槽",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "arrow",
			"version": 1724,
			"versionNonce": 1723067051,
			"isDeleted": false,
			"id": "J14jObGbx7PC25pLNKDCx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -437.80899628951045,
			"y": 1249.6959429793435,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 0.6212644144087562,
			"height": 62.10337421413237,
			"seed": 1388373379,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "rB1uWGMBnujG2mAKY5_0f",
				"focus": 0.02754128134355828,
				"gap": 8.599006758912537
			},
			"endBinding": {
				"elementId": "MOqctMbP1r4vY5TYSkUPy",
				"focus": 0.03677233930170947,
				"gap": 3.2875903977484313
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
					0.6212644144087562,
					62.10337421413237
				]
			]
		},
		{
			"type": "arrow",
			"version": 1923,
			"versionNonce": 2127931781,
			"isDeleted": false,
			"id": "gMQlwVkn7u7qVtj-kBHea",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -561.951096096313,
			"y": 555.9358238789773,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 48.26543391602047,
			"height": 65.52416224828414,
			"seed": 1447758787,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "EtmeMz-BdnDUhKY0TuO1E",
				"focus": 0.9023532869664974,
				"gap": 21.18458368157917
			},
			"endBinding": {
				"elementId": "EtmeMz-BdnDUhKY0TuO1E",
				"gap": 13.022155929631708,
				"focus": -0.012879454153245078
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
					-36.93654508579243,
					22.617354772430986
				],
				[
					-40.10300616407301,
					54.979090938519334
				],
				[
					-22.725266188044657,
					65.52416224828414
				],
				[
					8.162427751947462,
					57.89300786495653
				]
			]
		},
		{
			"type": "arrow",
			"version": 2009,
			"versionNonce": 1104633163,
			"isDeleted": false,
			"id": "1vXz6kIEeCJWzvTstvh13",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -340.4687436086299,
			"y": 801.6090868058149,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 341.13082318615085,
			"height": 1325.198804594369,
			"seed": 2062610755,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "QxfCD9js"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "iLXE5W5Lx1BfsXxEUTs4c",
				"focus": -0.8253695652242472,
				"gap": 13.331518427983369
			},
			"endBinding": {
				"elementId": "T3l9HpdHo4Bku8vxH8bYJ",
				"focus": 0.3915434430035143,
				"gap": 16.28834169360954
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
					301.46272026918086,
					450.5480438339928
				],
				[
					176.18736319656296,
					990.3738051317205
				],
				[
					-39.668102916969985,
					1325.198804594369
				]
			]
		},
		{
			"type": "text",
			"version": 500,
			"versionNonce": 1875914981,
			"isDeleted": false,
			"id": "QxfCD9js",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -90.03525932998082,
			"y": 1521.9295340620977,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 40.09596252441406,
			"height": 20,
			"seed": 507437325,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "false",
			"rawText": "false",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "1vXz6kIEeCJWzvTstvh13",
			"originalText": "false",
			"lineHeight": 1.25,
			"baseline": 14
		},
		{
			"type": "text",
			"version": 469,
			"versionNonce": 653619461,
			"isDeleted": false,
			"id": "vbIORLCP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 142.63962752147785,
			"y": 1928.2557121324753,
			"strokeColor": "#1971c2",
			"backgroundColor": "#ffc9c9",
			"width": 346.7198791503906,
			"height": 60,
			"seed": 1210843395,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243196026,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "往后未找到key相同的槽。若往后寻找过程\n发现脏槽，且staleSlot和slotToExpunge相等，\n则更新slotToExpunge为当前脏槽位置",
			"rawText": "往后未找到key相同的槽。若往后寻找过程\n发现脏槽，且staleSlot和slotToExpunge相等，\n则更新slotToExpunge为当前脏槽位置",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "往后未找到key相同的槽。若往后寻找过程\n发现脏槽，且staleSlot和slotToExpunge相等，\n则更新slotToExpunge为当前脏槽位置",
			"lineHeight": 1.25,
			"baseline": 54
		},
		{
			"type": "text",
			"version": 578,
			"versionNonce": 710789861,
			"isDeleted": false,
			"id": "qjdS1HD8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 807.9283146934695,
			"y": 2243.3668513873686,
			"strokeColor": "#1971c2",
			"backgroundColor": "#ffc9c9",
			"width": 426.7198791503906,
			"height": 40,
			"seed": 342474691,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693243029073,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "若slotToExpunge与staleSlot脏槽相同，\n代表staleSlot前无脏槽，更新slotToExpunge为当前槽位置",
			"rawText": "若slotToExpunge与staleSlot脏槽相同，\n代表staleSlot前无脏槽，更新slotToExpunge为当前槽位置",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "若slotToExpunge与staleSlot脏槽相同，\n代表staleSlot前无脏槽，更新slotToExpunge为当前槽位置",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "text",
			"version": 785,
			"versionNonce": 2027079307,
			"isDeleted": false,
			"id": "DynALBCf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 20.084428720270353,
			"y": 2573.6547317649342,
			"strokeColor": "#1971c2",
			"backgroundColor": "#fcc2d7",
			"width": 192,
			"height": 40,
			"seed": 545951373,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "相等代表往前与往后都没有\n找到脏槽，无需清理",
			"rawText": "相等代表往前与往后都没有\n找到脏槽，无需清理",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "相等代表往前与往后都没有\n找到脏槽，无需清理",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "diamond",
			"version": 1312,
			"versionNonce": 1200290725,
			"isDeleted": false,
			"id": "AQKgDex8OdsNgQmhIi8AO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 17.651956133250565,
			"y": 370.29263196094905,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 1928323877,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "btk8qdn0"
				},
				{
					"id": "vhbQTHolPNpk-tVDQvwAn",
					"type": "arrow"
				},
				{
					"id": "s_SRIRgRtSwtzD6gfnbRi",
					"type": "arrow"
				},
				{
					"id": "5F-vZ-l_CVHW3xe_7bq_7",
					"type": "arrow"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1409,
			"versionNonce": 124763435,
			"isDeleted": false,
			"id": "btk8qdn0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 92.65195613325056,
			"y": 463.69263196094903,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 80,
			"height": 19.2,
			"seed": 62011013,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "找到空槽？",
			"rawText": "找到空槽？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "AQKgDex8OdsNgQmhIi8AO",
			"originalText": "找到空槽？",
			"lineHeight": 1.2,
			"baseline": 14
		},
		{
			"id": "s_SRIRgRtSwtzD6gfnbRi",
			"type": "arrow",
			"x": 131.34550361039055,
			"y": 577.9333762325158,
			"width": 1.4634967683708169,
			"height": 102.23902146105377,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 198480907,
			"version": 531,
			"versionNonce": 911621893,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "KUPH1fNB"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-1.4634967683708169,
					102.23902146105377
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "AQKgDex8OdsNgQmhIi8AO",
				"focus": -0.001664553214456129,
				"gap": 2.0938247844472784
			},
			"endBinding": {
				"elementId": "1qyd9Up9kbbvRiMAmlteP",
				"focus": 0.011117860990794643,
				"gap": 3.4201096366346206
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "KUPH1fNB",
			"type": "text",
			"x": 107.97133211773325,
			"y": 607.6800406223961,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1817287627,
			"version": 66,
			"versionNonce": 812965835,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "s_SRIRgRtSwtzD6gfnbRi",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"id": "5F-vZ-l_CVHW3xe_7bq_7",
			"type": "arrow",
			"x": 11.757696888137616,
			"y": 455.4741721474332,
			"width": 331.09152691370707,
			"height": 84.16563283615443,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 718311275,
			"version": 479,
			"versionNonce": 1705121381,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "hUy6Oc7o"
				}
			],
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-156.87511020041507,
					-84.16563283615443
				],
				[
					-331.09152691370707,
					-55.35086532141315
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "AQKgDex8OdsNgQmhIi8AO",
				"focus": -0.45672773323778243,
				"gap": 17.205506751268416
			},
			"endBinding": {
				"elementId": "Ln77prd8BvzhNPGPbMSNP",
				"focus": 0.3317633607999788,
				"gap": 6.852707060217085
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "hUy6Oc7o",
			"type": "text",
			"x": -177.20801440179258,
			"y": 437.94369645424337,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"1ATtC1pyB-ixM1yYnl7vZ",
				"bLfm6_rfiz8LJB1jhmm0M"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1977242437,
			"version": 65,
			"versionNonce": 1308405355,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693242755478,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "5F-vZ-l_CVHW3xe_7bq_7",
			"originalText": "true",
			"lineHeight": 1.25
		}
	],
	"appState": {
		"theme": "light",
		"viewBackgroundColor": "#f5faff",
		"currentItemStrokeColor": "#1e1e1e",
		"currentItemBackgroundColor": "#ffc9c9",
		"currentItemFillStyle": "hachure",
		"currentItemStrokeWidth": 1,
		"currentItemStrokeStyle": "solid",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 1,
		"currentItemFontSize": 16,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": 4849.504118585075,
		"scrollY": 1881.5638359209606,
		"zoom": {
			"value": 0.1477482292056084
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