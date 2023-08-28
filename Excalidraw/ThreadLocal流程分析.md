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

从当前槽开始进行遍历，进行新值插入 ^KS9auWrS

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

从清理结果的后一个位置开始贪心清理其它的脏数据，按比例进行最大努力清理 ^3U6X046w

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

计算该过期数据的哈希槽，是否应位于当前位置？ ^9eXw2dWN

对该元素重新rehash放到合适的位置 ^86i4X2V2

true ^JW58vCOM

true ^zIPMfYVZ

false ^uZNRrXoC

true ^ZMts38zj

false ^o164zHVU

继续循环寻找 ^Pq9K9J1q

往后寻找其他过期数据，直到遇到空槽 ^GAmFZmDr

true ^bobj21M9

false ^QxfCD9js

未往后找到key相同的槽，若发现脏槽，
更新slotToExpunge为脏槽位置 ^vbIORLCP

若slotToExpunge与staleSlot脏槽相同，
代表staleSlot前无脏草，更新slotToExpunge为当前槽位置 ^qjdS1HD8

相等代表往前与往后都没有
找到脏槽，无需清理 ^DynALBCf

set(ThreadLocal<?> key, Object value) ^VCyv2XUy

%%
# Drawing
```json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/1.9.12",
	"elements": [
		{
			"type": "rectangle",
			"version": 126,
			"versionNonce": 1134320205,
			"isDeleted": false,
			"id": "H7JgZBK3-4ISXgXxlWm3m",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1222.9945100298896,
			"y": 717.117742683038,
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
			"updated": 1693156375469,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 126,
			"versionNonce": 380042499,
			"isDeleted": false,
			"id": "gM3Josfy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1230.726535176374,
			"y": 735.617742683038,
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
			"updated": 1693156375469,
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
			"version": 148,
			"versionNonce": 1226148013,
			"isDeleted": false,
			"id": "aGAMDMZzFxdbZDwhnh487",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1525.952843363223,
			"y": 613.6333676830382,
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
			"updated": 1693156375469,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 76,
			"versionNonce": 593912995,
			"isDeleted": false,
			"id": "pAElg1OscOdu7q3CPfO3k",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1565.8382600298896,
			"y": 720.758367683038,
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
			"updated": 1693156375469,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 66,
			"versionNonce": 2014964493,
			"isDeleted": false,
			"id": "ps6Ed9tQ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1572.570285176374,
			"y": 736.758367683038,
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
			"updated": 1693156375469,
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
			"version": 315,
			"versionNonce": 521656387,
			"isDeleted": false,
			"id": "Wve9Zl2h3TLB6iHEf4YAP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1870.1455516965564,
			"y": 724.5292010163713,
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
			"updated": 1693156375469,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 45,
			"versionNonce": 1629684077,
			"isDeleted": false,
			"id": "E3bidyFV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1912.8055629880603,
			"y": 750.5292010163713,
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
			"updated": 1693156375470,
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
			"version": 367,
			"versionNonce": 2006570979,
			"isDeleted": false,
			"id": "C9IWsQ5S5kBfX1-33AYT6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1997.1455516965561,
			"y": 724.5292010163713,
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
			"updated": 1693156375470,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 39,
			"versionNonce": 1232751565,
			"isDeleted": false,
			"id": "P7oNOlNE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2039.80556298806,
			"y": 750.5292010163713,
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
			"updated": 1693156375470,
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
			"version": 364,
			"versionNonce": 1526391683,
			"isDeleted": false,
			"id": "2YKxP6VP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2124.1455516965557,
			"y": 724.5292010163713,
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
			"updated": 1693156375470,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 39,
			"versionNonce": 288967213,
			"isDeleted": false,
			"id": "z8NljqPX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2166.8055629880596,
			"y": 750.5292010163713,
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
			"updated": 1693156375470,
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
			"version": 150,
			"versionNonce": 830044963,
			"isDeleted": false,
			"id": "G8U43p0jnnkjOdRhPrMCA",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1963.890343363223,
			"y": 854.992742683038,
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
			"updated": 1693156375470,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 128,
			"versionNonce": 1497988237,
			"isDeleted": false,
			"id": "huNKEHX6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1985.7943140739276,
			"y": 876.7188790756618,
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
			"updated": 1693156375470,
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
			"version": 146,
			"versionNonce": 1468276419,
			"isDeleted": false,
			"id": "Jst0KSo0UhAjsBiGaqcxt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1831.9997183632233,
			"y": 854.992742683038,
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
			"updated": 1693156375470,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 126,
			"versionNonce": 1768186605,
			"isDeleted": false,
			"id": "B1ytGnPD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1862.4236857169942,
			"y": 876.7188790756618,
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
			"updated": 1693156375470,
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
			"version": 230,
			"versionNonce": 162444835,
			"isDeleted": false,
			"id": "5U-sLAcAsOnAuCWThY0XB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1934.74451002989,
			"y": 805.7479510163714,
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
			"updated": 1693156746371,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Wve9Zl2h3TLB6iHEf4YAP",
				"gap": 9.218750000000114,
				"focus": -0.484604941213858
			},
			"endBinding": {
				"elementId": "Jst0KSo0UhAjsBiGaqcxt",
				"gap": 3.2160998429975365,
				"focus": -0.480065126119063
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
			"version": 256,
			"versionNonce": 1207664259,
			"isDeleted": false,
			"id": "4JbCfQNEU3UwKKPCiJM1q",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1936.38513502989,
			"y": 809.6021176830382,
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
			"updated": 1693156746369,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Wve9Zl2h3TLB6iHEf4YAP",
				"gap": 13.072916666666856,
				"focus": 0.6706244771618274
			},
			"endBinding": {
				"elementId": "G8U43p0jnnkjOdRhPrMCA",
				"gap": 5.358001979909378,
				"focus": 0.9785305201316687
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
			"version": 279,
			"versionNonce": 190446019,
			"isDeleted": false,
			"id": "RDUacjA2CC-7lW0fpnFQv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1822.78245745843,
			"y": 875.402471714177,
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
			"updated": 1693156746371,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Jst0KSo0UhAjsBiGaqcxt",
				"gap": 11.067242500482614,
				"focus": -0.7322907994375165
			},
			"endBinding": {
				"elementId": "pAElg1OscOdu7q3CPfO3k",
				"gap": 9.40625,
				"focus": -0.6447238426999354
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
			"version": 284,
			"versionNonce": 139534243,
			"isDeleted": false,
			"id": "UGI7mmj8s8OdZpBhMmh-J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1360.7080516965564,
			"y": 758.3104510163714,
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
			"updated": 1693156746360,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "H7JgZBK3-4ISXgXxlWm3m",
				"gap": 8.713541666666742,
				"focus": 0.06274246188457154
			},
			"endBinding": {
				"elementId": "pAElg1OscOdu7q3CPfO3k",
				"gap": 9.85416666666606,
				"focus": -0.06943918165456228
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
			"version": 168,
			"versionNonce": 1410159011,
			"isDeleted": false,
			"id": "CC4klGTp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1545.8590933632229,
			"y": 632.1906593497049,
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
			"updated": 1693156375470,
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
			"version": 140,
			"versionNonce": 996429325,
			"isDeleted": false,
			"id": "dRAX5KR9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1980.6299266965566,
			"y": 674.5864926830379,
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
			"updated": 1693156375470,
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
			"id": "bLeyZJ4P",
			"type": "text",
			"x": 1797.020060280924,
			"y": 941.8101262681062,
			"width": 123.16793823242188,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"nqN1OWaO0_Z4zF9yzL8Gn"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1791371981,
			"version": 119,
			"versionNonce": 1448682819,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156375470,
			"link": null,
			"locked": false,
			"text": "Weak Reference",
			"rawText": "Weak Reference",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 14,
			"containerId": null,
			"originalText": "Weak Reference",
			"lineHeight": 1.25
		},
		{
			"type": "text",
			"version": 180,
			"versionNonce": 858326125,
			"isDeleted": false,
			"id": "jmRpKkoh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1968.9740240071023,
			"y": 941.8101262681062,
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
			"updated": 1693156375470,
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
			"id": "nzVzk-m7P4EKtjHZsx9xH",
			"type": "arrow",
			"x": -983.5198492167826,
			"y": -283.53889321882815,
			"width": 406.74679478523933,
			"height": 2.121616287629081,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1355320557,
			"version": 506,
			"versionNonce": 2083388547,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157362454,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					406.74679478523933,
					-2.121616287629081
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "dPTtpbMtozWyadMgE4VZT",
				"focus": -0.15047506170708685,
				"gap": 10.045494915498239
			},
			"endBinding": {
				"elementId": "kPCsFHlJo8DVvxcUcuWLY",
				"focus": -0.07041955275244037,
				"gap": 9.873802433865535
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"type": "rectangle",
			"version": 703,
			"versionNonce": 1286693261,
			"isDeleted": false,
			"id": "kPCsFHlJo8DVvxcUcuWLY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -566.8992519976778,
			"y": -319.272689906732,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 327,
			"height": 61,
			"seed": 826969101,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1341,
			"versionNonce": 1652957773,
			"isDeleted": false,
			"id": "GC7AahB2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -555.9111691119356,
			"y": -298.37268990673203,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 305.0238342285156,
			"height": 19.2,
			"seed": 495543395,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 373,
			"versionNonce": 169719043,
			"isDeleted": false,
			"id": "EINmIFJwSEw6J5Ev_asxp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -488.37379043281635,
			"y": -153.90895089046865,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 183,
			"height": 77,
			"seed": 1056917891,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 339,
			"versionNonce": 758231821,
			"isDeleted": false,
			"id": "Tjrz95sq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -482.48176925361713,
			"y": -134.60895089046863,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 171.21595764160156,
			"height": 38.4,
			"seed": 1123155085,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 847,
			"versionNonce": 1044418627,
			"isDeleted": false,
			"id": "0oide05HgsqhqlWv_nDj4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -518.2183767137548,
			"y": 27.681761056309767,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 455658413,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 919,
			"versionNonce": 1633000323,
			"isDeleted": false,
			"id": "TgMckWqZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -452.80236108875476,
			"y": 92.28176105630976,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 99.16796875,
			"height": 76.8,
			"seed": 361831459,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1254,
			"versionNonce": 2084989453,
			"isDeleted": false,
			"id": "4ofT9kvGoGvogoTR78YrG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -405.4944548886604,
			"y": -249.33982996441821,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 2.0692054925930847,
			"height": 86.78506287329171,
			"seed": 1581054733,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693157329260,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "kPCsFHlJo8DVvxcUcuWLY",
				"gap": 8.932859942313797,
				"focus": 0.007033000908735407
			},
			"endBinding": {
				"elementId": "EINmIFJwSEw6J5Ev_asxp",
				"gap": 8.645816200657862,
				"focus": -0.12783189526656638
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
					-2.0692054925930847,
					86.78506287329171
				]
			]
		},
		{
			"type": "arrow",
			"version": 1750,
			"versionNonce": 1843004813,
			"isDeleted": false,
			"id": "oEJv1sJIZhPpVnOeyKwXH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -405.5788525538617,
			"y": -72.88303534945518,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 0.7641104239105516,
			"height": 97.84408800034906,
			"seed": 27142829,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "EINmIFJwSEw6J5Ev_asxp",
				"focus": 0.09844337188997138,
				"gap": 4.025915541013461
			},
			"endBinding": {
				"elementId": "0oide05HgsqhqlWv_nDj4",
				"focus": -0.006702111002039828,
				"gap": 3.0917148444838602
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
					0.7641104239105516,
					97.84408800034906
				]
			]
		},
		{
			"type": "rectangle",
			"version": 820,
			"versionNonce": 4015245,
			"isDeleted": false,
			"id": "Ln77prd8BvzhNPGPbMSNP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -502.237865730881,
			"y": 328.00481396860414,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"width": 214,
			"height": 111,
			"seed": 1004126947,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
				}
			],
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 667,
			"versionNonce": 242157155,
			"isDeleted": false,
			"id": "5fz1GrTr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -483.237865730881,
			"y": 364.30481396860415,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 176,
			"height": 38.4,
			"seed": 1003113389,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 815,
			"versionNonce": 414532941,
			"isDeleted": false,
			"id": "EtmeMz-BdnDUhKY0TuO1E",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -502.8178410598283,
			"y": 520.6143945607093,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"width": 201,
			"height": 106,
			"seed": 86109901,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 776,
			"versionNonce": 927587651,
			"isDeleted": false,
			"id": "0G2kiKfZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -496.6618275710588,
			"y": 554.4143945607093,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 188.68797302246094,
			"height": 38.4,
			"seed": 293151875,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 2005,
			"versionNonce": 217828333,
			"isDeleted": false,
			"id": "GPJdiyWt3gOVvOfzEnBOe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -397.25230309803226,
			"y": 230.84519329910083,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 2.0150614013339236,
			"height": 93.63591354573535,
			"seed": 197285091,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0oide05HgsqhqlWv_nDj4",
				"focus": -0.07705013525274838,
				"gap": 1.8674398074690686
			},
			"endBinding": {
				"elementId": "Ln77prd8BvzhNPGPbMSNP",
				"focus": 0.011699312965973704,
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
					0.5696223291224669,
					93.63591354573535
				]
			]
		},
		{
			"id": "6HeOGlaD",
			"type": "text",
			"x": -295.8135192486069,
			"y": 276.6353606891452,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 480528813,
			"version": 155,
			"versionNonce": 625163491,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "GPJdiyWt3gOVvOfzEnBOe",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"type": "arrow",
			"version": 2391,
			"versionNonce": 1154860621,
			"isDeleted": false,
			"id": "meWBtmUsoCkato1C6_gIt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -396.25537246104307,
			"y": 445.9780869949201,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 2.4411182759521353,
			"height": 63.72434369279546,
			"seed": 2024817731,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Ln77prd8BvzhNPGPbMSNP",
				"focus": -0.012606394286405134,
				"gap": 6.973273026315951
			},
			"endBinding": {
				"elementId": "EtmeMz-BdnDUhKY0TuO1E",
				"focus": 0.011440957849532361,
				"gap": 10.911963872993795
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
					63.72434369279546
				]
			]
		},
		{
			"type": "diamond",
			"version": 963,
			"versionNonce": 942647427,
			"isDeleted": false,
			"id": "T3l9HpdHo4Bku8vxH8bYJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -501.0280482754047,
			"y": 2060.6838000156617,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 181,
			"height": 212,
			"seed": 1742662403,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 892,
			"versionNonce": 671851459,
			"isDeleted": false,
			"id": "EWYM5bxy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -450.2780482754047,
			"y": 2118.6838000156617,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 80,
			"height": 96,
			"seed": 327694115,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 952,
			"versionNonce": 1791208941,
			"isDeleted": false,
			"id": "1LDn5lFf9YzfJmTwbovMv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -542.7397225265149,
			"y": 2416.498446924936,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 247,
			"height": 118,
			"seed": 1806852419,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 910,
			"versionNonce": 51498755,
			"isDeleted": false,
			"id": "3FFNqegp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -534.2236855392102,
			"y": 2456.2984469249363,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 229.96792602539062,
			"height": 38.4,
			"seed": 882174541,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1135,
			"versionNonce": 1257821869,
			"isDeleted": false,
			"id": "cEJ-yBHYHnZKZPLnmew3V",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -500.4337344330071,
			"y": 2767.5953506286846,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffa94d",
			"width": 156,
			"height": 125,
			"seed": 1137199075,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1071,
			"versionNonce": 926439885,
			"isDeleted": false,
			"id": "w32DwnMJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -438.5880633655578,
			"y": 2820.301176804525,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 32,
			"height": 19.2,
			"seed": 70266371,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 3078,
			"versionNonce": 943813805,
			"isDeleted": false,
			"id": "AYCf8n2KnLW4XAKa3FQ-R",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -411.8038909889433,
			"y": 2276.4825383012444,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.4977257983486538,
			"height": 131.33946071712535,
			"seed": 1683993069,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "T3l9HpdHo4Bku8vxH8bYJ",
				"focus": 0.0002624964611062554,
				"gap": 3.4368756724557556
			},
			"endBinding": {
				"elementId": "1LDn5lFf9YzfJmTwbovMv",
				"focus": 0.04160620597393283,
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
					-1.4977257983486538,
					131.33946071712535
				]
			]
		},
		{
			"id": "bobj21M9",
			"type": "text",
			"x": -429.080588173726,
			"y": 2309.8117595063572,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1880917827,
			"version": 381,
			"versionNonce": 2013632557,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "AYCf8n2KnLW4XAKa3FQ-R",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"type": "arrow",
			"version": 3602,
			"versionNonce": 1927829261,
			"isDeleted": false,
			"id": "PjGHh6io4PNZffjmIAfaD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -413.1688842623928,
			"y": 2546.3619337670402,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.5802721800981772,
			"height": 211.17040801604207,
			"seed": 749978605,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1LDn5lFf9YzfJmTwbovMv",
				"focus": -0.05326010125490858,
				"gap": 11.863486842104066
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.09155681716055865,
				"gap": 10.337736246116435
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
					-1.5802721800981772,
					211.17040801604207
				]
			]
		},
		{
			"type": "rectangle",
			"version": 555,
			"versionNonce": 2103184013,
			"isDeleted": false,
			"id": "uTogZR3U2ZowzQ3qqCj0J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 67.95908946544034,
			"y": 348.82611702516385,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"width": 229,
			"height": 87,
			"seed": 1670379395,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
					"id": "YWlIUUhZ08REye1Z-HVgl",
					"type": "arrow"
				},
				{
					"id": "vhbQTHolPNpk-tVDQvwAn",
					"type": "arrow"
				},
				{
					"id": "nPRV0tyFPsQrLngjh30PC",
					"type": "arrow"
				}
			],
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 492,
			"versionNonce": 89819981,
			"isDeleted": false,
			"id": "KS9auWrS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 78.45908946544034,
			"y": 373.12611702516386,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 208,
			"height": 38.4,
			"seed": 1158650605,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "从当前槽开始进行遍历，进行\n新值插入",
			"rawText": "从当前槽开始进行遍历，进行新值插入",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "uTogZR3U2ZowzQ3qqCj0J",
			"originalText": "从当前槽开始进行遍历，进行新值插入",
			"lineHeight": 1.2,
			"baseline": 33
		},
		{
			"type": "arrow",
			"version": 1328,
			"versionNonce": 1329851757,
			"isDeleted": false,
			"id": "YWlIUUhZ08REye1Z-HVgl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -283.78705613855107,
			"y": 133.37330027051576,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 493.8790218638998,
			"height": 205.77961964128184,
			"seed": 1675106573,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0oide05HgsqhqlWv_nDj4",
				"focus": -0.1912603531090624,
				"gap": 4.961393466141146
			},
			"endBinding": {
				"elementId": "uTogZR3U2ZowzQ3qqCj0J",
				"focus": 0.48132887435345867,
				"gap": 9.673197113366257
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
					379.03931828700394,
					71.06347074931301
				],
				[
					493.8790218638998,
					205.77961964128184
				]
			]
		},
		{
			"type": "text",
			"version": 191,
			"versionNonce": 1977981357,
			"isDeleted": false,
			"id": "xpwI8oVY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -10.090775847729105,
			"y": 21.39032569577344,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 1667887683,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1011,
			"versionNonce": 2015093667,
			"isDeleted": false,
			"id": "1qyd9Up9kbbvRiMAmlteP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 57.82041759044034,
			"y": 527.0897889001637,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 1624589731,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
				}
			],
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1091,
			"versionNonce": 1312528099,
			"isDeleted": false,
			"id": "aLA9nlcM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 120.79642314463956,
			"y": 591.6897889001639,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 104.04798889160156,
			"height": 76.8,
			"seed": 1845664067,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1199,
			"versionNonce": 480098253,
			"isDeleted": false,
			"id": "vhbQTHolPNpk-tVDQvwAn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 179.46885509044034,
			"y": 450.57416390016385,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.24609375,
			"height": 70.69531250000006,
			"seed": 167171661,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894199,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "uTogZR3U2ZowzQ3qqCj0J",
				"focus": 0.01703474299050022,
				"gap": 14.748046875
			},
			"endBinding": {
				"elementId": "1qyd9Up9kbbvRiMAmlteP",
				"focus": 0.030297816957915183,
				"gap": 7.939865076478952
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
					-1.24609375,
					70.69531250000006
				]
			]
		},
		{
			"type": "rectangle",
			"version": 759,
			"versionNonce": 353157763,
			"isDeleted": false,
			"id": "ssI7SygbNLwooDH7PriRx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 64.15970330472612,
			"y": 875.1941415787351,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 1543739853,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 705,
			"versionNonce": 1290726797,
			"isDeleted": false,
			"id": "EMab5nm1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 72.11571221585893,
			"y": 918.5941415787352,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.08798217773438,
			"height": 19.2,
			"seed": 1634261037,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 2357,
			"versionNonce": 84882989,
			"isDeleted": false,
			"id": "-ghrUwKzysCBvFwQM59lR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 160.53791517681015,
			"y": 992.0847665787358,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 530.3649397873967,
			"height": 1778.0051709471397,
			"seed": 950299597,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ssI7SygbNLwooDH7PriRx",
				"focus": -0.03798316345285455,
				"gap": 10.890625000000682
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.34072001221652515,
				"gap": 11.567148559390404
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
					-90.68966375847066,
					706.8375546380013
				],
				[
					-530.3649397873967,
					1778.0051709471397
				]
			]
		},
		{
			"type": "arrow",
			"version": 1524,
			"versionNonce": 864151693,
			"isDeleted": false,
			"id": "lmYszLTlsjoHrEsS2Pgkz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 167.94308093543026,
			"y": 739.4887920159072,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.1628801095705512,
			"height": 131.77956831282813,
			"seed": 446509091,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1qyd9Up9kbbvRiMAmlteP",
				"focus": 0.04123581756228159,
				"gap": 8.020662307984082
			},
			"endBinding": {
				"elementId": "ssI7SygbNLwooDH7PriRx",
				"focus": 0.030329852312558147,
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
					-0.1628801095705512,
					131.77956831282813
				]
			]
		},
		{
			"type": "text",
			"version": 356,
			"versionNonce": 469994851,
			"isDeleted": false,
			"id": "VJL8qAyN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 153.0169761566512,
			"y": 773.4643886441224,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 898436387,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1215,
			"versionNonce": 1104195149,
			"isDeleted": false,
			"id": "2M26vrb-1aSBNSmMVPVH8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 472.9185177324857,
			"y": 793.3535588986695,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 230,
			"height": 206,
			"seed": 695652973,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1295,
			"versionNonce": 1279868685,
			"isDeleted": false,
			"id": "XjTRT9XX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 543.894523286685,
			"y": 867.5535588986695,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 88.04798889160156,
			"height": 57.599999999999994,
			"seed": 1496054989,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1724,
			"versionNonce": 967790317,
			"isDeleted": false,
			"id": "2Jzxq1AsuCcE9fijlQZne",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 302.8026237812636,
			"y": 638.9945862763116,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 273.597795152089,
			"height": 152.19575121717162,
			"seed": 1030416771,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1qyd9Up9kbbvRiMAmlteP",
				"focus": -0.09897209883358246,
				"gap": 16.628935162534063
			},
			"endBinding": {
				"elementId": "2M26vrb-1aSBNSmMVPVH8",
				"focus": 0.6137333763871702,
				"gap": 9.295958560226367
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
					174.83890531485838,
					25.689938531449116
				],
				[
					273.597795152089,
					152.19575121717162
				]
			]
		},
		{
			"type": "text",
			"version": 357,
			"versionNonce": 474469741,
			"isDeleted": false,
			"id": "Ft2vYUcu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 457.59354783391495,
			"y": 655.0845248077605,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1064587245,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1748,
			"versionNonce": 908126541,
			"isDeleted": false,
			"id": "nPRV0tyFPsQrLngjh30PC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 665.9037135453717,
			"y": 845.2511937650725,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 353.951104903795,
			"height": 453.1874075936753,
			"seed": 257585517,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2M26vrb-1aSBNSmMVPVH8",
				"focus": 0.7482496102902082,
				"gap": 13.370979657321797
			},
			"endBinding": {
				"elementId": "uTogZR3U2ZowzQ3qqCj0J",
				"focus": -0.5957085791178869,
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
					-51.53633217652225,
					-326.6107030482208
				],
				[
					-353.951104903795,
					-453.1874075936753
				]
			]
		},
		{
			"type": "text",
			"version": 369,
			"versionNonce": 1894031309,
			"isDeleted": false,
			"id": "5Fhf6okP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 500.78339699384946,
			"y": 499.4404907168516,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 227.16796875,
			"height": 38.4,
			"seed": 293233251,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 844,
			"versionNonce": 1963970435,
			"isDeleted": false,
			"id": "VpvvT94PCb9K2NfVd2B10",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 490.784995005213,
			"y": 1132.1035588986715,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 201,
			"height": 106,
			"seed": 2041756077,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 793,
			"versionNonce": 19845261,
			"isDeleted": false,
			"id": "DebJ7p0F",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 498.38103382357235,
			"y": 1156.3035588986716,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.80792236328125,
			"height": 57.599999999999994,
			"seed": 1609104397,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 902,
			"versionNonce": 172886723,
			"isDeleted": false,
			"id": "CoQ_g_7z03BwYj7CzfHvD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 490.784995005213,
			"y": 1352.16037708049,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eebefa",
			"width": 201,
			"height": 106,
			"seed": 1891669325,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 861,
			"versionNonce": 1538511181,
			"isDeleted": false,
			"id": "AyW2Y6C4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 502.52503101595516,
			"y": 1385.96037708049,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 177.51992797851562,
			"height": 38.4,
			"seed": 1543136173,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 952,
			"versionNonce": 1193702915,
			"isDeleted": false,
			"id": "NSwFDKRVBEpY2lpoJzuRM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 490.784995005213,
			"y": 1554.7313998077625,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 201,
			"height": 106,
			"seed": 48093645,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 913,
			"versionNonce": 738173453,
			"isDeleted": false,
			"id": "upHm9JqX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 498.6850194192755,
			"y": 1588.5313998077625,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.199951171875,
			"height": 38.4,
			"seed": 855316525,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1177,
			"versionNonce": 1322062147,
			"isDeleted": false,
			"id": "qLcFjKvWDXMtoAljewKg8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 487.76937000521275,
			"y": 1741.14475208049,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 206,
			"seed": 988665613,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1259,
			"versionNonce": 125164675,
			"isDeleted": false,
			"id": "lB4IWJ2t",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 558.745375559412,
			"y": 1824.94475208049,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 88.04798889160156,
			"height": 38.4,
			"seed": 2056875373,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 995,
			"versionNonce": 952525101,
			"isDeleted": false,
			"id": "DvebHZK2RZWWrE3NkMmhg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 694.5278927324855,
			"y": 2066.414638444128,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 201,
			"height": 106,
			"seed": 44836803,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 951,
			"versionNonce": 1515509699,
			"isDeleted": false,
			"id": "AKE26X9p",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 702.427917146548,
			"y": 2090.6146384441276,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 185.199951171875,
			"height": 57.599999999999994,
			"seed": 318256995,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1100,
			"versionNonce": 1138211309,
			"isDeleted": false,
			"id": "LfZ6_qXkksKdwHzCBngwS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 694.5278927324855,
			"y": 2342.285006837104,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffd8a8",
			"width": 201,
			"height": 106,
			"seed": 155561421,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1069,
			"versionNonce": 528081581,
			"isDeleted": false,
			"id": "Iq4OQuY9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 706.2679287432277,
			"y": 2375.285006837104,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 177.51992797851562,
			"height": 40,
			"seed": 1162374189,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1111,
			"versionNonce": 395094691,
			"isDeleted": false,
			"id": "Bq0G5TkPGbYTHJM7woWyM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 694.5278927324855,
			"y": 2549.856876449537,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 128660579,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1067,
			"versionNonce": 1505287021,
			"isDeleted": false,
			"id": "3U6X046w",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 707.0278927324855,
			"y": 2564.4568764495366,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 176,
			"height": 76.8,
			"seed": 162312195,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 4,
			"text": "从清理结果的后一个位置\n开始贪心清理其它的脏数\n据，按比例进行最大努力\n清理",
			"rawText": "从清理结果的后一个位置开始贪心清理其它的脏数据，按比例进行最大努力清理",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "Bq0G5TkPGbYTHJM7woWyM",
			"originalText": "从清理结果的后一个位置开始贪心清理其它的脏数据，按比例进行最大努力清理",
			"lineHeight": 1.2,
			"baseline": 71
		},
		{
			"type": "rectangle",
			"version": 1247,
			"versionNonce": 1600560611,
			"isDeleted": false,
			"id": "FU9MnYQBBpelNIyU08ezs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 166.21552586950463,
			"y": 2079.091146709426,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#99e9f2",
			"width": 201,
			"height": 106,
			"seed": 2044517411,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1201,
			"versionNonce": 1839244333,
			"isDeleted": false,
			"id": "OdyXTW0c",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 214.11555028356713,
			"y": 2112.8911467094263,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 105.199951171875,
			"height": 38.4,
			"seed": 1590910915,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1587,
			"versionNonce": 894222627,
			"isDeleted": false,
			"id": "Cn0ySZnfSN_0EYNXnVud-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 145.83360685175376,
			"y": 2305.8877184085927,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 230,
			"height": 212,
			"seed": 51778147,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1663,
			"versionNonce": 1207112803,
			"isDeleted": false,
			"id": "L3nS944u",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 208.44964231317954,
			"y": 2363.8877184085927,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 104.76792907714844,
			"height": 96,
			"seed": 1342950915,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1537,
			"versionNonce": 976238509,
			"isDeleted": false,
			"id": "04KhpxuYYeLkMiMCbbsoe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 588.1558629412538,
			"y": 1004.1336495367518,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1.2029990265888273,
			"height": 110.73553436191878,
			"seed": 2075435555,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2M26vrb-1aSBNSmMVPVH8",
				"focus": -0.01224554038223928,
				"gap": 3.719051405404471
			},
			"endBinding": {
				"elementId": "VpvvT94PCb9K2NfVd2B10",
				"focus": -0.05040908239533355,
				"gap": 17.23437500000091
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
					-1.2029990265888273,
					110.73553436191878
				]
			]
		},
		{
			"type": "text",
			"version": 356,
			"versionNonce": 905482243,
			"isDeleted": false,
			"id": "Zqon31a5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 570.6558809218294,
			"y": 1051.6036784292319,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 450371139,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1106,
			"versionNonce": 1879489037,
			"isDeleted": false,
			"id": "Pq6dqDtDC3xGQ9RJLzCDM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 586.7395404597585,
			"y": 1248.5055475350346,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.9232954545453822,
			"height": 89.40340909090901,
			"seed": 707367181,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "VpvvT94PCb9K2NfVd2B10",
				"focus": 0.03850356867339114,
				"gap": 10.401988636363058
			},
			"endBinding": {
				"elementId": "CoQ_g_7z03BwYj7CzfHvD",
				"focus": -0.060993935190292455,
				"gap": 14.251420454546405
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
			"version": 1173,
			"versionNonce": 10350701,
			"isDeleted": false,
			"id": "oMfxFNtmZ9ABrTviZrzJM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 593.01958431009,
			"y": 1468.8352724831686,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.1404106951230233,
			"height": 74.59215005186593,
			"seed": 1811474573,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CoQ_g_7z03BwYj7CzfHvD",
				"focus": -0.016051021619845986,
				"gap": 10.674895402678658
			},
			"endBinding": {
				"elementId": "NSwFDKRVBEpY2lpoJzuRM",
				"focus": 0.019841442307801733,
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
			"version": 1141,
			"versionNonce": 968841933,
			"isDeleted": false,
			"id": "kqR6RUHJxSRRMLm9BAPvx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 592.4639722779402,
			"y": 1675.0254338986717,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.06392045454549589,
			"height": 64.98579545454459,
			"seed": 564617187,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894200,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "NSwFDKRVBEpY2lpoJzuRM",
				"focus": -0.01106676115984289,
				"gap": 14.294034090909236
			},
			"endBinding": {
				"elementId": "qLcFjKvWDXMtoAljewKg8",
				"focus": -0.08816565962547514,
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
					64.98579545454459
				]
			]
		},
		{
			"type": "arrow",
			"version": 1217,
			"versionNonce": 1946450221,
			"isDeleted": false,
			"id": "PChAFJtfnID7nKNMxa3bd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 621.6116995506676,
			"y": 1956.1191838986715,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 178.77840909090878,
			"height": 97.07386363636306,
			"seed": 2071562851,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qLcFjKvWDXMtoAljewKg8",
				"focus": 0.9877994494962,
				"gap": 19.256177852453305
			},
			"endBinding": {
				"elementId": "DvebHZK2RZWWrE3NkMmhg",
				"focus": 0.6426814341830497,
				"gap": 13.221590909093266
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
					178.77840909090878,
					97.07386363636306
				]
			]
		},
		{
			"type": "text",
			"version": 356,
			"versionNonce": 885317229,
			"isDeleted": false,
			"id": "vEUtpMHM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 694.1209144720987,
			"y": 1995.0561157168531,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 1388191341,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1639,
			"versionNonce": 965147619,
			"isDeleted": false,
			"id": "BB36ZpmNOq3D6C71EsqCZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 587.8196523855619,
			"y": 1956.8171538443676,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 351.7670872828951,
			"height": 110.24467116062533,
			"seed": 1735162093,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693157178788,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qLcFjKvWDXMtoAljewKg8",
				"focus": -1.0484186743452553,
				"gap": 17.179050000101228
			},
			"endBinding": {
				"elementId": "FU9MnYQBBpelNIyU08ezs",
				"focus": -0.8833356869992862,
				"gap": 12.029321704433187
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
					-351.7670872828951,
					110.24467116062533
				]
			]
		},
		{
			"type": "text",
			"version": 357,
			"versionNonce": 1725466829,
			"isDeleted": false,
			"id": "21n3w6UH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 422.7675535157332,
			"y": 1973.9197520804892,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1173447875,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1316,
			"versionNonce": 678416877,
			"isDeleted": false,
			"id": "HXqJt1_2265Ap2VnR2h-z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 789.7897591505517,
			"y": 2184.1518543532206,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 2.048109491428022,
			"height": 151.55218657479236,
			"seed": 956240781,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "DvebHZK2RZWWrE3NkMmhg",
				"focus": 0.06039550268359846,
				"gap": 11.73721590909281
			},
			"endBinding": {
				"elementId": "LfZ6_qXkksKdwHzCBngwS",
				"focus": -0.023561767631863968,
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
					2.048109491428022,
					151.55218657479236
				]
			]
		},
		{
			"type": "arrow",
			"version": 1358,
			"versionNonce": 534318157,
			"isDeleted": false,
			"id": "2DgazU-DKk52e1D_qwETa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 796.9683916223213,
			"y": 2458.495234109831,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 0.7833980776404132,
			"height": 82.39431279425071,
			"seed": 1315720163,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "LfZ6_qXkksKdwHzCBngwS",
				"focus": -0.013261876007534427,
				"gap": 10.210227272727025
			},
			"endBinding": {
				"elementId": "Bq0G5TkPGbYTHJM7woWyM",
				"focus": 0.0328014700154386,
				"gap": 8.967329545455414
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
			"version": 2900,
			"versionNonce": 518781613,
			"isDeleted": false,
			"id": "4XIY0Qdaz4mUGbX4I3hHt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 797.2957939918732,
			"y": 2673.4782331229303,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 1130.8952919711974,
			"height": 175.8197256693179,
			"seed": 250464493,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Bq0G5TkPGbYTHJM7woWyM",
				"focus": -0.5490380824285882,
				"gap": 17.621356673393166
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.3799464128978409,
				"gap": 13.770254974280434
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
					-160.07374929896866,
					125.62054157320517
				],
				[
					-1130.8952919711974,
					175.8197256693179
				]
			]
		},
		{
			"type": "arrow",
			"version": 2546,
			"versionNonce": 1850875149,
			"isDeleted": false,
			"id": "gWYpx7rL6q0SJw2hdApu2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 174.26347170033972,
			"y": 2470.41146811674,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 523.981306406407,
			"height": 331.4284548148953,
			"seed": 427700675,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Cn0ySZnfSN_0EYNXnVud-",
				"focus": -0.03553246272945684,
				"gap": 23.76382674958772
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": 0.2227757853619745,
				"gap": 2.6783489735375525
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
					-523.981306406407,
					331.4284548148953
				]
			]
		},
		{
			"type": "text",
			"version": 288,
			"versionNonce": 1330344387,
			"isDeleted": false,
			"id": "IraALWd2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -39.16584215081761,
			"y": 2389.975662885144,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 33.759979248046875,
			"height": 19.2,
			"seed": 2053958701,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 2028,
			"versionNonce": 92599149,
			"isDeleted": false,
			"id": "YQPb_DuNh1mKFEvRj_EBI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 382.80588418478015,
			"y": 2405.920616856795,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 300.2178549864232,
			"height": 1.8136449238136265,
			"seed": 2063412589,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Cn0ySZnfSN_0EYNXnVud-",
				"focus": -0.049342031117574014,
				"gap": 9.113023723847377
			},
			"endBinding": {
				"elementId": "LfZ6_qXkksKdwHzCBngwS",
				"focus": -0.15194503440076806,
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
					300.2178549864232,
					-1.8136449238136265
				]
			]
		},
		{
			"type": "text",
			"version": 355,
			"versionNonce": 1629372771,
			"isDeleted": false,
			"id": "GZfGxkwO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 518.112277541795,
			"y": 2325.103288737451,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1939684429,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"version": 1853,
			"versionNonce": 818806221,
			"isDeleted": false,
			"id": "y-DKJ7x2coeUHulAPtXx6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 261.107103790506,
			"y": 2192.2159437873484,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 3.2980037866558405,
			"height": 109.64967039519797,
			"seed": 120191533,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "FU9MnYQBBpelNIyU08ezs",
				"focus": 0.0726470309269076,
				"gap": 7.124797077922267
			},
			"endBinding": {
				"elementId": "Cn0ySZnfSN_0EYNXnVud-",
				"focus": 0.05983222995098544,
				"gap": 5.378007513837147
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
					3.2980037866558405,
					109.64967039519797
				]
			]
		},
		{
			"type": "arrow",
			"version": 2634,
			"versionNonce": 2001124397,
			"isDeleted": false,
			"id": "xJj1W6f_5J-goSJcBq56-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -509.15744012384664,
			"y": 2173.3509852386346,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 245.68573558775438,
			"height": 652.1649920544992,
			"seed": 63986093,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156894201,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "T3l9HpdHo4Bku8vxH8bYJ",
				"focus": 0.6842439638077323,
				"gap": 10.511665592333785
			},
			"endBinding": {
				"elementId": "cEJ-yBHYHnZKZPLnmew3V",
				"focus": -0.9800424498311755,
				"gap": 7.922289096015248
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
					-244.70334992440416,
					196.4909439976475
				],
				[
					0.9823856633502146,
					652.1649920544992
				]
			]
		},
		{
			"type": "text",
			"version": 475,
			"versionNonce": 1351116973,
			"isDeleted": false,
			"id": "RID7hBEU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -699.9330424377223,
			"y": 1681.9526452839873,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 40.09596252441406,
			"height": 19.2,
			"seed": 1337919139,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"id": "pxmNdqXn",
			"type": "text",
			"x": -794.9235775922873,
			"y": 553.9852267234498,
			"width": 211.615966796875,
			"height": 60,
			"angle": 0,
			"strokeColor": "#1971c2",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1250806509,
			"version": 882,
			"versionNonce": 689013923,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"text": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0。即log(n)的扫描次数",
			"rawText": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0。即log(n)的扫描次数",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 54,
			"containerId": null,
			"originalText": "按比例回收，回收次数为\n当前哈希表长度处以2，\n直到为0。即log(n)的扫描次数",
			"lineHeight": 1.25
		},
		{
			"type": "text",
			"version": 454,
			"versionNonce": 1311820557,
			"isDeleted": false,
			"id": "C2CbM31X",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 936.5016757168859,
			"y": 2579.430203941655,
			"strokeColor": "#1971c2",
			"backgroundColor": "#fcc2d7",
			"width": 176,
			"height": 60,
			"seed": 1940926893,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"id": "1bezDJzh",
			"type": "text",
			"x": 306.89814134878645,
			"y": 347.58780482708534,
			"width": 144,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1971c2",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 39648227,
			"version": 494,
			"versionNonce": 39778371,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"text": "使用开放地址法寻址",
			"rawText": "使用开放地址法寻址",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 14,
			"containerId": null,
			"originalText": "使用开放地址法寻址",
			"lineHeight": 1.25
		},
		{
			"type": "diamond",
			"version": 894,
			"versionNonce": 291365229,
			"isDeleted": false,
			"id": "iLXE5W5Lx1BfsXxEUTs4c",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -481.65954214453404,
			"y": 678.1048342932654,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 181,
			"height": 183,
			"seed": 1986888067,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 839,
			"versionNonce": 1480337965,
			"isDeleted": false,
			"id": "PEtP0Pni",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -430.90954214453404,
			"y": 750.6548342932654,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 80,
			"height": 38.4,
			"seed": 1088968995,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893834,
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
			"id": "t8JA2zsR",
			"type": "text",
			"x": -309.3567430433005,
			"y": 703.517014588622,
			"width": 184.04798889160156,
			"height": 40,
			"angle": 0,
			"strokeColor": "#1971c2",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 794496013,
			"version": 625,
			"versionNonce": 2023004963,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893834,
			"link": null,
			"locked": false,
			"text": "过期数据：key已经被回收\n但value未被回收",
			"rawText": "过期数据：key已经被回收\n但value未被回收",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 34,
			"containerId": null,
			"originalText": "过期数据：key已经被回收\n但value未被回收",
			"lineHeight": 1.25
		},
		{
			"type": "rectangle",
			"version": 1099,
			"versionNonce": 2096127117,
			"isDeleted": false,
			"id": "UazpQQDxoCA1Irisu8SLu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -493.0495933605964,
			"y": 932.1341651414339,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 1398348707,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
				}
			],
			"updated": 1693156893834,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1187,
			"versionNonce": 2054152973,
			"isDeleted": false,
			"id": "xZppoh6a",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -456.5495933605964,
			"y": 975.5341651414338,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 128,
			"height": 19.2,
			"seed": 1302048579,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693157013225,
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
			"version": 1005,
			"versionNonce": 625159085,
			"isDeleted": false,
			"id": "MOqctMbP1r4vY5TYSkUPy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -492.12902154526887,
			"y": 1302.8929601834525,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"width": 181,
			"height": 183,
			"seed": 1314723117,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893835,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 997,
			"versionNonce": 1384375405,
			"isDeleted": false,
			"id": "VnPM0VWB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -441.37902154526887,
			"y": 1365.8429601834525,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 80,
			"height": 57.599999999999994,
			"seed": 1975248781,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893835,
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
			"version": 1451,
			"versionNonce": 1684279597,
			"isDeleted": false,
			"id": "ed0-Q7RYXL6OhosPGyes7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -491.8472046283545,
			"y": 1570.5941089267817,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 176,
			"height": 242,
			"seed": 19986979,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693157115955,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1523,
			"versionNonce": 276997155,
			"isDeleted": false,
			"id": "9eXw2dWN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -442.34718936956546,
			"y": 1645.3718867045595,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 76.99996948242188,
			"height": 92.4444444444444,
			"seed": 1643363779,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693157115955,
			"link": null,
			"locked": false,
			"fontSize": 15.407407407407401,
			"fontFamily": 4,
			"text": "计算该过期\n数据的哈希\n槽，是否应\n位于当前位\n置？",
			"rawText": "计算该过期数据的哈希槽，是否应位于当前位置？",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "ed0-Q7RYXL6OhosPGyes7",
			"originalText": "计算该过期数据的哈希槽，是否应位于当前位置？",
			"lineHeight": 1.2,
			"baseline": 86
		},
		{
			"type": "rectangle",
			"version": 1266,
			"versionNonce": 361840525,
			"isDeleted": false,
			"id": "uyYqxenN6jX5fD1o9M88_",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -505.85402284027487,
			"y": 1892.8989298190018,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 201,
			"height": 106,
			"seed": 1883649325,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
			"updated": 1693156893835,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1408,
			"versionNonce": 826121059,
			"isDeleted": false,
			"id": "86i4X2V2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -494.7540014779702,
			"y": 1926.6989298190017,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 178.79995727539062,
			"height": 38.4,
			"seed": 1031866253,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893835,
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
			"id": "XvLzY_87WT1etuKK5faAW",
			"type": "arrow",
			"x": -395.89283276206913,
			"y": 633.1698375940844,
			"width": 0.47144300109005144,
			"height": 40.37593913660953,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1798138403,
			"version": 1455,
			"versionNonce": 1087942285,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156894202,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-0.47144300109005144,
					40.37593913660953
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "jpMeRYHlOZfoOBOvnr2w4",
			"type": "arrow",
			"x": -391.0530178664585,
			"y": 862.2554106668745,
			"width": 0.44859501925964196,
			"height": 68.87875447455929,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1549583245,
			"version": 1542,
			"versionNonce": 1506697187,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "JW58vCOM"
				}
			],
			"updated": 1693157013832,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.44859501925964196,
					68.87875447455929
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "iLXE5W5Lx1BfsXxEUTs4c",
				"gap": 1.1554970416780606,
				"focus": 0.005490523416443588
			},
			"endBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"gap": 1.0000000000001137,
				"focus": 0.02277612945237117
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "JW58vCOM",
			"type": "text",
			"x": -289.97919859548006,
			"y": 892.5898285517305,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 541429933,
			"version": 368,
			"versionNonce": 1355476653,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893835,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "jpMeRYHlOZfoOBOvnr2w4",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"id": "P1nU3PADse2jEjjXpEaDu",
			"type": "arrow",
			"x": -394.19561487981696,
			"y": 1044.8193176544578,
			"width": 1.790600138554339,
			"height": 74.49885000974632,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1068048483,
			"version": 1734,
			"versionNonce": 358509443,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157013843,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.790600138554339,
					74.49885000974632
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"gap": 6.685152513023922,
				"focus": 0.030268775507039075
			},
			"endBinding": {
				"elementId": "rB1uWGMBnujG2mAKY5_0f",
				"gap": 4.613668440299307,
				"focus": 0.0655746728006502
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "wyBVBCs-6Gjk6ejaoSxWT",
			"type": "arrow",
			"x": -400.4404102072874,
			"y": 1491.1129202573802,
			"width": 2.464383054869984,
			"height": 78.90679972663202,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 77034403,
			"version": 2438,
			"versionNonce": 1803782253,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "zIPMfYVZ"
				}
			],
			"updated": 1693157115953,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-2.464383054869984,
					78.90679972663202
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "MOqctMbP1r4vY5TYSkUPy",
				"gap": 5.353576382772221,
				"focus": -0.048291178792153715
			},
			"endBinding": {
				"elementId": "ed0-Q7RYXL6OhosPGyes7",
				"gap": 1.1000872021449086,
				"focus": -0.03250369229431972
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "zIPMfYVZ",
			"type": "text",
			"x": -292.5169884603263,
			"y": 1326.4693148342922,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 35126115,
			"version": 374,
			"versionNonce": 1321505347,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893835,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "wyBVBCs-6Gjk6ejaoSxWT",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"id": "eGKZYb_A7u4UWOToPz_nz",
			"type": "arrow",
			"x": -406.1729128857307,
			"y": 1814.9881770000097,
			"width": 0.6982898309341863,
			"height": 71.64004622283505,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 2029831021,
			"version": 2256,
			"versionNonce": 1680995021,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "uZNRrXoC"
				}
			],
			"updated": 1693157115954,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.6982898309341863,
					71.64004622283505
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "ed0-Q7RYXL6OhosPGyes7",
				"gap": 3.3377358849491245,
				"focus": 0.04072200121004238
			},
			"endBinding": {
				"elementId": "uyYqxenN6jX5fD1o9M88_",
				"gap": 6.270706596156963,
				"focus": 0.004525228396346846
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "uZNRrXoC",
			"type": "text",
			"x": -298.44959243613755,
			"y": 1647.0286490091726,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1742141091,
			"version": 383,
			"versionNonce": 1168069091,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893835,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "eGKZYb_A7u4UWOToPz_nz",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"id": "Gs6pFPMYMJ0IKqH0wKXUd",
			"type": "arrow",
			"x": -332.3139215039246,
			"y": 1643.6154464116921,
			"width": 121.470479250817,
			"height": 615.6243000499894,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1310277389,
			"version": 2046,
			"versionNonce": 641019011,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "ZMts38zj"
				}
			],
			"updated": 1693157115954,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					121.470479250817,
					-411.1319562707222
				],
				[
					42.231053326652955,
					-615.6243000499894
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "ed0-Q7RYXL6OhosPGyes7",
				"gap": 14.999689990526631,
				"focus": 0.6531124776397947
			},
			"endBinding": {
				"elementId": "UazpQQDxoCA1Irisu8SLu",
				"gap": 1.9667251833247974,
				"focus": -0.709368410284025
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "ZMts38zj",
			"type": "text",
			"x": -227.72343187713102,
			"y": 1222.48349014097,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 984133261,
			"version": 375,
			"versionNonce": 1345671939,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157120959,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "Gs6pFPMYMJ0IKqH0wKXUd",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"id": "ImqDWpKayT1X0uRagq8S7",
			"type": "arrow",
			"x": -489.0400304970692,
			"y": 1379.940711038436,
			"width": 174.19120718160713,
			"height": 743.94877550721,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1602269261,
			"version": 1839,
			"versionNonce": 1279953357,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "o164zHVU"
				}
			],
			"updated": 1693157044395,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-159.15717599804805,
					-446.4867211542504
				],
				[
					15.03403118355908,
					-743.94877550721
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "MOqctMbP1r4vY5TYSkUPy",
				"focus": -0.9089423057718681,
				"gap": 7.9667653377101
			},
			"endBinding": {
				"elementId": "EtmeMz-BdnDUhKY0TuO1E",
				"focus": 0.2673056670433881,
				"gap": 9.37754097051652
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "o164zHVU",
			"type": "text",
			"x": -562.0931733822653,
			"y": 834.0040709813538,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 204949357,
			"version": 376,
			"versionNonce": 1139307811,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893835,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "ImqDWpKayT1X0uRagq8S7",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"id": "TG5lRPxWyBKybCqK_JuLq",
			"type": "arrow",
			"x": -333.6299384430825,
			"y": 1889.1640170747792,
			"width": 217.99383395002627,
			"height": 892.8392887822636,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1836936269,
			"version": 2527,
			"versionNonce": 788672589,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "Pq9K9J1q"
				}
			],
			"updated": 1693157132362,
			"link": null,
			"locked": false,
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
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "Pq9K9J1q",
			"type": "text",
			"x": -8.710486934898313,
			"y": 1225.5901952463114,
			"width": 96,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 897973869,
			"version": 490,
			"versionNonce": 250443245,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157131812,
			"link": null,
			"locked": false,
			"text": "继续循环寻找",
			"rawText": "继续循环寻找",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "TG5lRPxWyBKybCqK_JuLq",
			"originalText": "继续循环寻找",
			"lineHeight": 1.25
		},
		{
			"type": "rectangle",
			"version": 1176,
			"versionNonce": 1062553837,
			"isDeleted": false,
			"id": "rB1uWGMBnujG2mAKY5_0f",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -498.19404272517704,
			"y": 1123.9318361045034,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"width": 201,
			"height": 106,
			"seed": 1545418051,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
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
				}
			],
			"updated": 1693156893835,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1284,
			"versionNonce": 507560963,
			"isDeleted": false,
			"id": "GAmFZmDr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -485.69404272517704,
			"y": 1157.7318361045034,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffec99",
			"width": 176,
			"height": 38.4,
			"seed": 99325155,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893835,
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
			"id": "J14jObGbx7PC25pLNKDCx",
			"type": "arrow",
			"x": -399.86032493460493,
			"y": 1238.530842863416,
			"width": 0.6212644144087562,
			"height": 62.10337421413237,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1388373379,
			"version": 1494,
			"versionNonce": 578202947,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157033896,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.6212644144087562,
					62.10337421413237
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "rB1uWGMBnujG2mAKY5_0f",
				"gap": 8.599006758912537,
				"focus": 0.02754128134355828
			},
			"endBinding": {
				"elementId": "MOqctMbP1r4vY5TYSkUPy",
				"gap": 3.2875903977484313,
				"focus": 0.03677233930170947
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "gMQlwVkn7u7qVtj-kBHea",
			"type": "arrow",
			"x": -524.0024247414075,
			"y": 544.7707237630498,
			"width": 48.26543391602047,
			"height": 65.52416224828414,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1447758787,
			"version": 1771,
			"versionNonce": 748542403,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156894202,
			"link": null,
			"locked": false,
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
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "1vXz6kIEeCJWzvTstvh13",
			"type": "arrow",
			"x": -302.5200722537244,
			"y": 790.4439866898872,
			"width": 341.13082318615085,
			"height": 1325.1988045943694,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 2062610755,
			"version": 1783,
			"versionNonce": 937666915,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "QxfCD9js"
				}
			],
			"updated": 1693156894202,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					301.46272026918086,
					450.5480438339929
				],
				[
					176.18736319656296,
					990.3738051317206
				],
				[
					-39.668102916969985,
					1325.1988045943694
				]
			],
			"lastCommittedPoint": null,
			"startBinding": {
				"elementId": "iLXE5W5Lx1BfsXxEUTs4c",
				"focus": -0.825369565224248,
				"gap": 13.331518427983298
			},
			"endBinding": {
				"elementId": "T3l9HpdHo4Bku8vxH8bYJ",
				"focus": 0.3915434430035175,
				"gap": 16.28834169360924
			},
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "QxfCD9js",
			"type": "text",
			"x": -52.086587975075304,
			"y": 1510.7644339461701,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#d0bfff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 507437325,
			"version": 426,
			"versionNonce": 938652941,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157121965,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "1vXz6kIEeCJWzvTstvh13",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"id": "vbIORLCP",
			"type": "text",
			"x": 198.10773602608708,
			"y": 1930.5917405900868,
			"width": 280.0479736328125,
			"height": 40,
			"angle": 0,
			"strokeColor": "#1971c2",
			"backgroundColor": "#ffc9c9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1210843395,
			"version": 266,
			"versionNonce": 242674285,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693156893835,
			"link": null,
			"locked": false,
			"text": "未往后找到key相同的槽，若发现脏槽，\n更新slotToExpunge为脏槽位置",
			"rawText": "未往后找到key相同的槽，若发现脏槽，\n更新slotToExpunge为脏槽位置",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 34,
			"containerId": null,
			"originalText": "未往后找到key相同的槽，若发现脏槽，\n更新slotToExpunge为脏槽位置",
			"lineHeight": 1.25
		},
		{
			"type": "text",
			"version": 499,
			"versionNonce": 1754595043,
			"isDeleted": false,
			"id": "qjdS1HD8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 845.876986048375,
			"y": 2232.2017512714406,
			"strokeColor": "#1971c2",
			"backgroundColor": "#ffc9c9",
			"width": 426.7198791503906,
			"height": 40,
			"seed": 342474691,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893835,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 1,
			"text": "若slotToExpunge与staleSlot脏槽相同，\n代表staleSlot前无脏草，更新slotToExpunge为当前槽位置",
			"rawText": "若slotToExpunge与staleSlot脏槽相同，\n代表staleSlot前无脏草，更新slotToExpunge为当前槽位置",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "若slotToExpunge与staleSlot脏槽相同，\n代表staleSlot前无脏草，更新slotToExpunge为当前槽位置",
			"lineHeight": 1.25,
			"baseline": 34
		},
		{
			"type": "text",
			"version": 711,
			"versionNonce": 1449727181,
			"isDeleted": false,
			"id": "DynALBCf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 58.03310007517587,
			"y": 2562.4896316490062,
			"strokeColor": "#1971c2",
			"backgroundColor": "#fcc2d7",
			"width": 192,
			"height": 40,
			"seed": 545951373,
			"groupIds": [
				"BeP_jD0HU3hWz0dHVRsIu"
			],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1693156893835,
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
			"id": "3ZMH1REJOxCMAMINO_pfk",
			"type": "rectangle",
			"x": -1878.7346820657983,
			"y": -1471.6296600234068,
			"width": 342,
			"height": 91,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"seed": 1635912867,
			"version": 593,
			"versionNonce": 2122700035,
			"isDeleted": false,
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
			"updated": 1693157349602,
			"link": null,
			"locked": false
		},
		{
			"id": "4o9RGpSy",
			"type": "text",
			"x": -1758.6626682108179,
			"y": -1436.1296600234068,
			"width": 101.85597229003906,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 287060963,
			"version": 546,
			"versionNonce": 1966092461,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"text": "get()方法调用",
			"rawText": "get()方法调用",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "3ZMH1REJOxCMAMINO_pfk",
			"originalText": "get()方法调用",
			"lineHeight": 1.25
		},
		{
			"id": "rrFtUuUP6_mP8M8MIZmpv",
			"type": "rectangle",
			"x": -1827.490496315547,
			"y": -1311.5312267289432,
			"width": 221,
			"height": 76,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#b2f2bb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"seed": 1838152013,
			"version": 547,
			"versionNonce": 1504328867,
			"isDeleted": false,
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
			"updated": 1693157349602,
			"link": null,
			"locked": false
		},
		{
			"id": "EbSLFXqj",
			"type": "text",
			"x": -1822.318476357051,
			"y": -1293.5312267289432,
			"width": 210.6559600830078,
			"height": 40,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1615921667,
			"version": 543,
			"versionNonce": 446665485,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"text": "从当前线程中获取ThreadLoc\nalMap对象",
			"rawText": "从当前线程中获取ThreadLocalMap对象",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 34,
			"containerId": "rrFtUuUP6_mP8M8MIZmpv",
			"originalText": "从当前线程中获取ThreadLocalMap对象",
			"lineHeight": 1.25
		},
		{
			"id": "F3rDIQBQ90d_UwRkiO7fb",
			"type": "diamond",
			"x": -1776.992017257334,
			"y": -1168.5766135238282,
			"width": 126,
			"height": 143,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1851961571,
			"version": 636,
			"versionNonce": 253800515,
			"isDeleted": false,
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
			"updated": 1693157349602,
			"link": null,
			"locked": false
		},
		{
			"id": "ggZ5eXgo",
			"type": "text",
			"x": -1737.992017257334,
			"y": -1116.8266135238282,
			"width": 48,
			"height": 40,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1118635501,
			"version": 579,
			"versionNonce": 1957931373,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"text": "map\n为空？",
			"rawText": "map\n为空？",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 34,
			"containerId": "F3rDIQBQ90d_UwRkiO7fb",
			"originalText": "map\n为空？",
			"lineHeight": 1.25
		},
		{
			"id": "4oCzCd9Krg_X3lI8yMOl0",
			"type": "rectangle",
			"x": -1804.9480656828016,
			"y": -959.125223839718,
			"width": 166,
			"height": 98,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#a5d8ff",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"seed": 338026755,
			"version": 619,
			"versionNonce": 549957603,
			"isDeleted": false,
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
			"updated": 1693157349602,
			"link": null,
			"locked": false
		},
		{
			"id": "xFSBzjWj",
			"type": "text",
			"x": -1796.3800343717664,
			"y": -930.125223839718,
			"width": 148.8639373779297,
			"height": 40,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#fcc2d7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1575446115,
			"version": 642,
			"versionNonce": 1958081485,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"text": "获取map中该Thread\nLocal对应Entry",
			"rawText": "获取map中该ThreadLocal对应Entry",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 34,
			"containerId": "4oCzCd9Krg_X3lI8yMOl0",
			"originalText": "获取map中该ThreadLocal对应Entry",
			"lineHeight": 1.25
		},
		{
			"type": "diamond",
			"version": 644,
			"versionNonce": 1353563011,
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
			"updated": 1693157349602,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 599,
			"versionNonce": 650113581,
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
			"updated": 1693157349602,
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
			"version": 686,
			"versionNonce": 454120227,
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
			"updated": 1693157349602,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 725,
			"versionNonce": 1471882381,
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
			"updated": 1693157349602,
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
			"id": "ubW48w_bRcfzgpGLeODeU",
			"type": "arrow",
			"x": -1708.0895490271953,
			"y": -1371.1875740954392,
			"width": 1.444645372584091,
			"height": 56.618339863898655,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 327187725,
			"version": 1536,
			"versionNonce": 1885402819,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.444645372584091,
					56.618339863898655
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "ey02iPLor1z17jcG9oMyd",
			"type": "arrow",
			"x": -1714.7834106892856,
			"y": -1227.3277721092568,
			"width": 1.2820677727449947,
			"height": 53.68017979510205,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 2097284835,
			"version": 1568,
			"versionNonce": 2069957357,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.2820677727449947,
					53.68017979510205
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "qVo6qizrTwtBC0ATga6Rv",
			"type": "arrow",
			"x": -1715.8111779558922,
			"y": -1022.0001870367776,
			"width": 1.027545428989015,
			"height": 54.50700356860784,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 733890979,
			"version": 1506,
			"versionNonce": 792405603,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "VezdmPpz"
				}
			],
			"updated": 1693157349602,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.027545428989015,
					54.50700356860784
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "VezdmPpz",
			"type": "text",
			"x": -1735.3355598493138,
			"y": -1004.2254221867313,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 535233645,
			"version": 483,
			"versionNonce": 1379607885,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "qVo6qizrTwtBC0ATga6Rv",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"id": "r87qwyjW46bnkcpVEsn43",
			"type": "arrow",
			"x": -1722.9080807893401,
			"y": -848.5913101135814,
			"width": 44.32383797827106,
			"height": 59.39205836179667,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 215319523,
			"version": 1687,
			"versionNonce": 1584292355,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-44.32383797827106,
					59.39205836179667
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "-pjGzbh_m4zg0UDmFRUyz",
			"type": "arrow",
			"x": -1768.3333793072252,
			"y": -637.1749770633262,
			"width": 2.312416402437293,
			"height": 86.8987368378954,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1703442595,
			"version": 1722,
			"versionNonce": 1336079277,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "TJ3IpQQy"
				}
			],
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-2.312416402437293,
					86.8987368378954
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "TJ3IpQQy",
			"type": "text",
			"x": -1788.2846315337026,
			"y": -619.9883947235097,
			"width": 40.09596252441406,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#e9ecef",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1621916515,
			"version": 483,
			"versionNonce": 653863331,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"text": "false",
			"rawText": "false",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "-pjGzbh_m4zg0UDmFRUyz",
			"originalText": "false",
			"lineHeight": 1.25
		},
		{
			"type": "rectangle",
			"version": 778,
			"versionNonce": 1069676045,
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
			"updated": 1693157349603,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 823,
			"versionNonce": 235535683,
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
			"updated": 1693157349603,
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
			"id": "ZLC0KhN3UaUIIhUvPuS-a",
			"type": "arrow",
			"x": -1641.9161601213837,
			"y": -1101.0522983809017,
			"width": 330.8857119287743,
			"height": 331.3327462397193,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 926551181,
			"version": 1514,
			"versionNonce": 764521581,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "RmiAXw8L"
				}
			],
			"updated": 1693157349603,
			"link": null,
			"locked": false,
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
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "RmiAXw8L",
			"type": "text",
			"x": -1449.558817199332,
			"y": -970.0305822266516,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1716560749,
			"version": 482,
			"versionNonce": 1862229219,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "ZLC0KhN3UaUIIhUvPuS-a",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"id": "3mID0RnAa0KP0QnjcQHRS",
			"type": "arrow",
			"x": -1694.8816683984596,
			"y": -712.0262333630111,
			"width": 286.3586553089201,
			"height": 1.9778822721377765,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1472355523,
			"version": 1548,
			"versionNonce": 148608717,
			"isDeleted": false,
			"boundElements": [
				{
					"type": "text",
					"id": "I0LZ5LMZ"
				}
			],
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					286.3586553089201,
					-1.9778822721377765
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "I0LZ5LMZ",
			"type": "text",
			"x": -1565.2459346448702,
			"y": -723.0382190197582,
			"width": 33.759979248046875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1528814179,
			"version": 482,
			"versionNonce": 895993987,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"text": "true",
			"rawText": "true",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "center",
			"verticalAlign": "middle",
			"baseline": 14,
			"containerId": "3mID0RnAa0KP0QnjcQHRS",
			"originalText": "true",
			"lineHeight": 1.25
		},
		{
			"type": "diamond",
			"version": 710,
			"versionNonce": 205075757,
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
			"updated": 1693157349603,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 650,
			"versionNonce": 1459732515,
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
			"updated": 1693157349603,
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
			"version": 931,
			"versionNonce": 1662257037,
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
			"updated": 1693157349603,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1094,
			"versionNonce": 230409155,
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
			"updated": 1693157349603,
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
			"id": "IxQqd5fo",
			"type": "text",
			"x": -1885.961510526633,
			"y": -283.2982161809547,
			"width": 296.623779296875,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1971c2",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 1458372877,
			"version": 691,
			"versionNonce": 262508013,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"text": "new ThreadLocalMap(this, firstValue);",
			"rawText": "new ThreadLocalMap(this, firstValue);",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 14,
			"containerId": null,
			"originalText": "new ThreadLocalMap(this, firstValue);",
			"lineHeight": 1.25
		},
		{
			"type": "rectangle",
			"version": 1037,
			"versionNonce": 1370772323,
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
			"updated": 1693157349603,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 1294,
			"versionNonce": 284641357,
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
			"updated": 1693157349603,
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
			"id": "urZD-SopW_l7tUwgUEhbq",
			"type": "arrow",
			"x": -1315.53674350094,
			"y": -653.0098441484805,
			"width": 0.36012743018022775,
			"height": 74.44972281003709,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 881238979,
			"version": 1533,
			"versionNonce": 322563843,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.36012743018022775,
					74.44972281003709
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "FOTVwoYx1mYSRgjRCqryH",
			"type": "arrow",
			"x": -1357.2591001153057,
			"y": -457.92812469308205,
			"width": 129.68082290280813,
			"height": 121.37120904958715,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1219059043,
			"version": 1497,
			"versionNonce": 1222817453,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-129.68082290280813,
					121.37120904958715
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "048ebS9AuEaZk_nxxcD-d",
			"type": "arrow",
			"x": -1277.2889052056244,
			"y": -468.69784413775085,
			"width": 208.01954545825015,
			"height": 131.3724107524455,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 201246339,
			"version": 1670,
			"versionNonce": 885724835,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					208.01954545825015,
					131.3724107524455
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"type": "rectangle",
			"version": 825,
			"versionNonce": 571562253,
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
			"updated": 1693157349603,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 895,
			"versionNonce": 1159088707,
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
			"updated": 1693157349603,
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
			"id": "eaDWYep_WO-SrA58Rs3pw",
			"type": "arrow",
			"x": -1504.3295960931455,
			"y": -216.6261524159902,
			"width": 221.7999046766745,
			"height": 133.4743135954817,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1455901123,
			"version": 1657,
			"versionNonce": 1979494253,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					221.7999046766745,
					133.4743135954817
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "PYVheSZQcMZH9AOK1majW",
			"type": "arrow",
			"x": -1069.8661631623675,
			"y": -212.63657943937665,
			"width": 218.2496716982257,
			"height": 129.056386467695,
			"angle": 0,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#eaddd7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"UZTih65JjujPpSdiYnKHp",
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"seed": 1117690883,
			"version": 1664,
			"versionNonce": 2029652451,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-218.2496716982257,
					129.056386467695
				]
			],
			"lastCommittedPoint": null,
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
			"startArrowhead": null,
			"endArrowhead": "arrow"
		},
		{
			"id": "VCyv2XUy",
			"type": "text",
			"x": -1046.9111377361555,
			"y": -361.1259559988274,
			"width": 305.0238342285156,
			"height": 20,
			"angle": 0,
			"strokeColor": "#1971c2",
			"backgroundColor": "#ffc9c9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"groupIds": [
				"ifW3uW_NDSrGwW8qEef41"
			],
			"frameId": null,
			"roundness": null,
			"seed": 264141709,
			"version": 33,
			"versionNonce": 854473165,
			"isDeleted": false,
			"boundElements": null,
			"updated": 1693157349603,
			"link": null,
			"locked": false,
			"text": "set(ThreadLocal<?> key, Object value)",
			"rawText": "set(ThreadLocal<?> key, Object value)",
			"fontSize": 16,
			"fontFamily": 1,
			"textAlign": "left",
			"verticalAlign": "top",
			"baseline": 14,
			"containerId": null,
			"originalText": "set(ThreadLocal<?> key, Object value)",
			"lineHeight": 1.25
		}
	],
	"appState": {
		"theme": "light",
		"viewBackgroundColor": "#f5faff",
		"currentItemStrokeColor": "#1971c2",
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
		"scrollX": 2233.999818201663,
		"scrollY": 1168.5075469661506,
		"zoom": {
			"value": 0.7643535682559013
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