---
title: 何为物联网中的物模型
date: 2022/4/9 19:46:25
tags:
    - 物联网
    - 物模型
categories: 技术
---


## 物模型简单说明

**什么是物模型？**

物模型指将物理空间中的实体数字化，并在云端构建该实体的数据模型。在物联网平台中，定义物模型即定义产品功能。完成功能定义后，系统将自动生成该产品的物模型。物模型描述产品是什么、能做什么、可以对外提供哪些服务。

物模型TSL（Thing Specification Language）。是一个JSON格式的文件。它是物理空间中的实体，如传感器、车载装置、楼宇、工厂等在云端的数字化表示，从属性、功能和事件三个维度，分别描述了该实体是什么、能做什么、可以对外提供哪些信息。定义了这三个维度，即完成了产品功能的定义。


**物模型可分为:**

1. 属性(properties)：一般是设备的运行状态，如当前温度等
2. 功能(function)：设备可被调用的方法，支持定义参数，如执行某项任务
3. 事件(event)：是设备上报的通知，如告警或者发生故障，需要被及时处理，事件可以被订阅和推送。
4. 标签(tags)：是给设备打上特殊的标签，后面方便过滤筛选等操作


**设备模型使用场景:**

1. 前端通过模型定义动态展示设备运行状态或者设备操作界面,如动态展示设备属性和设备事件等等
2. 服务端可通过统一的API获取设备模型并进行相关操作,如: 在发送设备消息时进行参数校验,在收到设备消息进行类型转换处理.
3. 服务端可根据物模型进行建模，进行建立超级表或者数据库表，也或者是表字段等等
4. 服务器端逻辑处理等，比如告警，对设备进行动作输出


**数据结构:**

``` bash
    {
        "id":"设备ID",
        "name":"设备名称",
        "properties":[...属性],
        "functions":[...功能],
        "events":[...事件],
        "tags":[...标签]
    }
```


## 属性

用于定义设备属性,运行状态等如: 当前温度,当前CPU使用率等.
设备通过事件上报属性.

数据结构:

``` bash
    {
       "id": "temperature",    //属性标识(必填)
       "name": "温度",          //属性名称（必填）
       "valueType": {    //值类型
         "type": "float",    //类型标识,见类型表（必填）
         "min":0,    //取值范围最小值（非必填）
         "max":100,    //取值范围最大值（非必填）
         "unit":"celsiusDegrees",    //单位（非必填）
         "expands":{"key1":"value1"} //其他自定义拓展定义，相当于Map
       },
       "expands":{"key1":"value1"}， //其他自定义拓展定义，相当于Map
       "description": ""  //描述（非必填）
     }
```

## 功能

用于定义设备功能,平台可主动调用,例如: 播放语音,进行报警等等.

数据结构:

``` bash
    {
       "id": "playVoice", //功能标识（必填）
       "name": "播放声音", //功能名称（必填）
       "inputs": [  //输入参数
         {
           "id": "f1",    //参数标识（必填）
           "name": "开关",//参数名称（必填）
           "valueType": { //参数类型
             "type": "string"， //类型标识,见类型表（必填）
             "min": 0,    //取值范围最小值（非必填）
             "max": 1     //取值范围最大值（非必填）
           },
           "expands":{"key1":"value1"} //其他自定义拓展定义，相当于Map
         }
       ],
       "output": { //输出（非必填）
         "type": "boolean",//类型标识,见类型表（必填）
         "trueText": "1",
         "trueValue": "开",
         "falseText": "0",
         "falseValue": "关"
       },
       "expands":{"key1":"value1"} //其他自定义拓展定义，相当于Map
     }
```

## 事件

用于定义设备事件, 如: 定时上报设备属性, 设备报警等.

数据结构:

``` bash
    {
       "id": "fire_alarm", //事件标识（必填）
       "name": "火警",   //时间名称（必填）
       "valueType": {
         "type": "object",  //对象(结构体)类型（必填）
         "properties": [    //对象属性(结构与属性定义相同)（必填）
           {
             "id": "location", //参数标识（必填）
             "name": "地点", //参数名称（必填）
             "valueType": {
               "type": "string" //参数类型（必填）
             },
             "description": "描述"
           },
           {
             "id": "lng",
             "name": "经度",
             "valueType": {
               "type": "double"
             },
             "expands":{"gis":"lng"} //其他自定义拓展定义，相当于Map
           },
           {
             "id": "lat",
             "name": "纬度",
             "valueType": {
               "type": "double"
             },
              "expands":{"gis":"lat"} //其他自定义拓展定义，相当于Map
           }
         ]
       },
       "expands":{//其他自定义拓展定义，相当于Map
          "eventType": "event",  //时间类型
          "level": "urgent"  //事件级别：urgent紧急；warn警告；ordinary普通
       } 
    }
```


## 数据类型

所有类型共有属性:

- id 唯一标识
- name 名称
- description 描述
- expands 自定义配置

### 数字类型

1. int 整型
2. long 长整型
3. float 单精度浮点型
4. double 双精度浮点型
    以上均为数字类型,共有属性:
    - max 最大值
    - min 最小值
    - unit 单位
    
     例:

``` bash
        {
		  "id": "speed",
		  "name": "风速",
		  "valueType": {
			"type": "int",
			"min": 0,
			"max": 100,
			"unit": "turnPerSeconds"
		  },
		  "expands": {
			"readOnly": "true"
		  }
		}
		或者
		{
		  "id": "temperature",
		  "name": "温度",
		  "valueType": {
			"type": "float",
			"min": "0",
			"max": "100",
			"scale": 2,
			"unit": "celsiusDegrees"
		  },
		  "expands": {
			"readOnly": "true"
		  }
		}
``` 

### boolean 布尔类型
     属性
      - trueText 为true时的文本,默认为`是`
      - falseText 为false时的文本,默认为`否`
      - trueValue  为true时的值,默认为`true`
      - falseValue 为false时的值,默认为`false`

      例:
``` bash
		{
		  "id": "IsAutoCreateDir",
		  "name": "是否自动创建目录",
		  "valueType": {
			"type": "boolean",
			"trueText": "1",
			"trueValue": "自动创建目录",
			"falseText": "0",
			"falseValue": "不自动创建目录"
		  },
		  "description": "描述一翻",
		  "expands": {
			"readOnly": "false"
		  }
		}
``` 

### string 字符类型
      例:

``` bash
        {
           "type":"string",
           "expands":{"maxLength":"255"}
        }
		或者
		{
		  "id": "RealtimeSnapUrl",
		  "name": "实时预览图地址",
		  "valueType": {
			"type": "string",
			"expands": {
			  "maxLength": "100"
			}
		  },
		  "expands": {
			"readOnly": "false"
		  },
		  "description": "实时预览图仅设备在线时可用"
		}
``` 

### enum 枚举类型
    属性:
     - elements (Element)枚举中的元素

    Element:
     - value 枚举值
     - text 枚举文本
     - description 说明

    例:
``` bash
        {
            "type":"enum",
            "elements":[
                {"value":"1","text":"正常"},
                {"value":"-1","text":"警告"},
                {"value":"0","text":"未知"}
            ]
        }
		或者
		{
		  "id": "mode",
		  "name": "模式",
		  "valueType": {
			"type": "enum",
			"elements": [
			  {
				"text": "制冷",
				"value": "0"
			  },
			  {
				"value": "1",
				"text": "制热"
			  },
			  {
				"value": "2",
				"text": "通风"
			  }
			]
		  },
		  "expands": {
			"readOnly": "true"
		  },
		  "description": "0-制冷 1-制热 2-通风"
		}
``` 

### date 时间类型

    属性:
      - format 格式,如: yyyy-MM-dd
      - tz 时区,如: Asia/Shanghai
    
    例:
``` bash
        {
            "type":"date",
            "format":"yyyy-MM-dd",
            "tz": "Asia/Shanghai"
        }
		或者
		{
		  "id": "time",
		  "name": "时间",
		  "valueType": {
			"type": "date",
			"format": "yyyy-MM-dd HH:mm:ss"
		  },
		  "expands": {
			"readOnly": "false"
		  },
		  "description": "当前时间"
		}
``` 

### array 数组(集合)类型

    属性:
      - elementType 元素类型
      
    例:
``` bash
        {
            "type":"array",
            "elementType":{ 
                "type":"string"
            }
        }
		或者
		{
		  "id": "array",
		  "name": "数组",
		  "valueType": {
			"type": "array",
			"elementType": "double",
			"elementNumber": "3"
		  },
		  "expands": {
			"readOnly": "false"
		  },
		  "description": "数组"
		}
``` 

### object 对象(结构体)类型

     属性:
        - properties 属性列表
        
     例:

``` bash
        {
            "type":"object",
            "properties":[
                {
                 "id": "location",
                 "name": "地点",
                 "valueType": {
                   "type": "string"
                 }
               },
               {
                 "id": "lng",
                 "name": "经度",
                 "valueType": {
                   "type": "double"
                 },
                 "expands":{"gis":"lng"}
               },
               {
                 "id": "lat",
                 "name": "纬度",
                 "valueType": {
                   "type": "double"
                 },
                  "expands":{"gis":"lat"}
               }
            ]
        }
``` 

### geoPoint Geo地理位置类型(先不做)

支持以逗号分割当经纬度字符串以及map类型.
默认支持3种格式转换: 逗号分割字符:`145.1214,126.123` ,json格式:`{"lat":145.1214,"lon":126.123}`.

      例:
``` bash
        {
          "type":"geoPoint"
        }

``` 



## 数据单位

``` bash

    {
      "result": "000",
      "desc": "操作成功！",
      "data": [
        {
          "code": "percent",
          "type": "常用单位",
          "name": "百分比",
          "symbol": "%",
          "desc": "百分比(%)"
        },
        {
          "code": "count",
          "type": "常用单位",
          "name": "次",
          "symbol": "count",
          "desc": "次"
        },
        {
          "code": "turnPerSeconds",
          "type": "常用单位",
          "name": "转每分钟",
          "symbol": "r/min",
          "desc": "转每分钟"
        },
        {
          "code": "nanometer",
          "type": "长度单位",
          "name": "纳米",
          "symbol": "nm",
          "desc": "长度单位:纳米(nm)"
        },
        {
          "code": "micron",
          "type": "长度单位",
          "name": "微米",
          "symbol": "μm",
          "desc": "长度单位:微米(μm)"
        },
        {
          "code": "millimeter",
          "type": "长度单位",
          "name": "毫米",
          "symbol": "mm",
          "desc": "长度单位:毫米(mm)"
        },
        {
          "code": "centimeter",
          "type": "长度单位",
          "name": "厘米",
          "symbol": "cm",
          "desc": "长度单位:厘米(cm)"
        },
        {
          "code": "meter",
          "type": "长度单位",
          "name": "米",
          "symbol": "m",
          "desc": "长度单位:米(m)"
        },
        {
          "code": "kilometer",
          "type": "长度单位",
          "name": "千米",
          "symbol": "km",
          "desc": "长度单位:千米(km)"
        },
        {
          "code": "squareMillimeter",
          "type": "面积单位",
          "name": "平方毫米",
          "symbol": "mm²",
          "desc": "面积单位:平方毫米(mm²)"
        },
        {
          "code": "squareCentimeter",
          "type": "面积单位",
          "name": "平方厘米",
          "symbol": "cm²",
          "desc": "面积单位:平方厘米(cm²)"
        },
        {
          "code": "squareMeter",
          "type": "面积单位",
          "name": "平方米",
          "symbol": "m²",
          "desc": "面积单位:平方米(m²)"
        },
        {
          "code": "squareKilometer",
          "type": "面积单位",
          "name": "平方千米",
          "symbol": "km²",
          "desc": "面积单位:平方千米(km²)"
        },
        {
          "code": "hectare",
          "type": "面积单位",
          "name": "公顷",
          "symbol": "hm²",
          "desc": "面积单位:公顷(hm²)"
        },
        {
          "code": "days",
          "type": "时间单位",
          "name": "天",
          "symbol": "d",
          "desc": "时间单位:天(d)"
        },
        {
          "code": "hour",
          "type": "时间单位",
          "name": "小时",
          "symbol": "h",
          "desc": "时间单位:小时(h)"
        },
        {
          "code": "minutes",
          "type": "时间单位",
          "name": "分钟",
          "symbol": "min",
          "desc": "时间单位:分钟(m)"
        },
        {
          "code": "seconds",
          "type": "时间单位",
          "name": "秒",
          "symbol": "s",
          "desc": "时间单位:秒(s)"
        },
        {
          "code": "milliseconds",
          "type": "时间单位",
          "name": "毫秒",
          "symbol": "ms",
          "desc": "时间单位:毫秒(ms)"
        },
        {
          "code": "microseconds",
          "type": "时间单位",
          "name": "微秒",
          "symbol": "μs",
          "desc": "时间单位:微秒(μs)"
        },
        {
          "code": "nanoseconds",
          "type": "时间单位",
          "name": "纳秒",
          "symbol": "ns",
          "desc": "时间单位:纳秒(ns)"
        },
        {
          "code": "cubicMillimeter",
          "type": "体积单位",
          "name": "立方毫米",
          "symbol": "mm³",
          "desc": "体积单位:立方毫米(mm³)"
        },
        {
          "code": "cubicCentimeter",
          "type": "体积单位",
          "name": "立方厘米",
          "symbol": "cm³",
          "desc": "体积单位:立方厘米(cm³)"
        },
        {
          "code": "cubicMeter",
          "type": "体积单位",
          "name": "立方米",
          "symbol": "m³",
          "desc": "体积单位:立方米(m³)"
        },
        {
          "code": "cubicKilometer",
          "type": "体积单位",
          "name": "立方千米",
          "symbol": "km³",
          "desc": "体积单位:立方千米(km³)"
        },
        {
          "code": "cubicMeterPerSec",
          "type": "流量单位",
          "name": "立方米每秒",
          "symbol": "m³/s",
          "desc": "流量单位:立方米每秒(m³/s)"
        },
        {
          "code": "cubicKilometerPerSec",
          "type": "流量单位",
          "name": "立方千米每秒",
          "symbol": "km³/s",
          "desc": "流量单位:立方千米每秒(km³/s)"
        },
        {
          "code": "cubicCentimeterPerSec",
          "type": "流量单位",
          "name": "立方厘米每秒",
          "symbol": "cm³/s",
          "desc": "流量单位:立方厘米每秒(cm³/s)"
        },
        {
          "code": "litrePerSec",
          "type": "流量单位",
          "name": "升每秒",
          "symbol": "l/s",
          "desc": "流量单位:升每秒(l/s)"
        },
        {
          "code": "cubicMeterPerHour",
          "type": "流量单位",
          "name": "立方米每小时",
          "symbol": "m³/h",
          "desc": "流量单位:立方米每小时(m³/h)"
        },
        {
          "code": "cubicKilometerPerHour",
          "type": "流量单位",
          "name": "立方千米每小时",
          "symbol": "km³/h",
          "desc": "流量单位:立方千米每小时(km³/h)"
        },
        {
          "code": "cubicCentimeterPerHour",
          "type": "流量单位",
          "name": "立方厘米每小时",
          "symbol": "cm³/h",
          "desc": "流量单位:立方厘米每小时(cm³/h)"
        },
        {
          "code": "litrePerHour",
          "type": "流量单位",
          "name": "升每小时",
          "symbol": "l/h",
          "desc": "流量单位:升每小时(l/h)"
        },
        {
          "code": "milliliter",
          "type": "容积单位",
          "name": "毫升",
          "symbol": "mL",
          "desc": "容积单位:毫升(mL)"
        },
        {
          "code": "litre",
          "type": "容积单位",
          "name": "升",
          "symbol": "L",
          "desc": "容积单位:升(L)"
        },
        {
          "code": "milligram",
          "type": "质量单位",
          "name": "毫克",
          "symbol": "mg",
          "desc": "重量单位:毫克(mg)"
        },
        {
          "code": "gramme",
          "type": "质量单位",
          "name": "克",
          "symbol": "g",
          "desc": "重量单位:克(g)"
        },
        {
          "code": "kilogram",
          "type": "质量单位",
          "name": "千克",
          "symbol": "kg",
          "desc": "重量单位:千克(kg)"
        },
        {
          "code": "ton",
          "type": "质量单位",
          "name": "吨",
          "symbol": "t",
          "desc": "重量单位:吨(t)"
        },
        {
          "code": "pascal",
          "type": "压力单位",
          "name": "帕斯卡",
          "symbol": "Pa",
          "desc": "压力单位:帕斯卡(Pa)"
        },
        {
          "code": "kiloPascal",
          "type": "压力单位",
          "name": "千帕斯卡",
          "symbol": "kPa",
          "desc": "压力单位:千帕斯卡(kPa)"
        },
        {
          "code": "newton",
          "type": "力单位",
          "name": "牛顿",
          "symbol": "N",
          "desc": "力单位:牛顿(N)"
        },
        {
          "code": "newtonMeter",
          "type": "力单位",
          "name": "牛·米",
          "symbol": "N.m",
          "desc": "力单位:牛·米(N.m)"
        },
        {
          "code": "kelvin",
          "type": "温度单位",
          "name": "开尔文",
          "symbol": "K",
          "desc": "温度单位:开尔文(K)"
        },
        {
          "code": "celsiusDegrees",
          "type": "温度单位",
          "name": "摄氏度",
          "symbol": "℃",
          "desc": "温度单位:摄氏度(℃)"
        },
        {
          "code": "fahrenheit",
          "type": "温度单位",
          "name": "华氏度",
          "symbol": "℉",
          "desc": "温度单位:华氏度(℉)"
        },
        {
          "code": "joule",
          "type": "能量单位",
          "name": "焦耳",
          "symbol": "J",
          "desc": "能单位:焦耳(J)"
        },
        {
          "code": "cal",
          "type": "能量单位",
          "name": "卡",
          "symbol": "cal",
          "desc": "能单位:卡(cal)"
        },
        {
          "code": "watt",
          "type": "功率单位",
          "name": "瓦特",
          "symbol": "W",
          "desc": "功率单位:瓦特(W)"
        },
        {
          "code": "kilowatt",
          "type": "功率单位",
          "name": "千瓦特",
          "symbol": "kW",
          "desc": "功率单位:千瓦特(kW)"
        },
        {
          "code": "radian",
          "type": "角度单位",
          "name": "弧度",
          "symbol": "rad",
          "desc": "角度单位:弧度(rad)"
        },
        {
          "code": "degrees",
          "type": "角度单位",
          "name": "度",
          "symbol": "°",
          "desc": "角度单位:度(°)"
        },
        {
          "code": "fen",
          "type": "角度单位",
          "name": "[角]分",
          "symbol": "′",
          "desc": "角度单位:分(′)"
        },
        {
          "code": "angleSeconds",
          "type": "角度单位",
          "name": "[角]秒",
          "symbol": "″",
          "desc": "角度单位:度(″)"
        },
        {
          "code": "hertz",
          "type": "频率单位",
          "name": "赫兹",
          "symbol": "Hz",
          "desc": "频率单位:赫兹(Hz)"
        },
        {
          "code": "megahertz",
          "type": "频率单位",
          "name": "兆赫兹",
          "symbol": "MHz",
          "desc": "频率单位:兆赫兹(MHz)"
        },
        {
          "code": "ghertz",
          "type": "频率单位",
          "name": "G赫兹",
          "symbol": "GHz",
          "desc": "频率单位:G赫兹(GHz)"
        },
        {
          "code": "mPerSec",
          "type": "速度单位",
          "name": "米每秒",
          "symbol": "m/s",
          "desc": "速度单位:米每秒(m/s)"
        },
        {
          "code": "kmPerHr",
          "type": "速度单位",
          "name": "千米每小时",
          "symbol": "km/h",
          "desc": "速度单位:千米每小时(km/h)"
        },
        {
          "code": "knots",
          "type": "速度单位",
          "name": "节",
          "symbol": "kn",
          "desc": "速度单位:节(kn)"
        },
        {
          "code": "volt",
          "type": "电力单位",
          "name": "伏特",
          "symbol": "V",
          "desc": "电压:伏特(V)"
        },
        {
          "code": "kiloVolt",
          "type": "电力单位",
          "name": "千伏",
          "symbol": "kV",
          "desc": "电压:千伏(kV)"
        },
        {
          "code": "milliVolt",
          "type": "电力单位",
          "name": "毫伏",
          "symbol": "mV",
          "desc": "电压:毫伏(mV)"
        },
        {
          "code": "microVolt",
          "type": "电力单位",
          "name": "微伏",
          "symbol": "μV",
          "desc": "电压:微伏(μV)"
        },
        {
          "code": "ampere",
          "type": "电力单位",
          "name": "安培",
          "symbol": "A",
          "desc": "电流:安培(A)"
        },
        {
          "code": "milliAmpere",
          "type": "电力单位",
          "name": "毫安",
          "symbol": "mA",
          "desc": "电流:毫安(mA)"
        },
        {
          "code": "microAmpere",
          "type": "电力单位",
          "name": "微安",
          "symbol": "μA",
          "desc": "电流:微安(μA)"
        },
        {
          "code": "nanoAmpere",
          "type": "电力单位",
          "name": "纳安",
          "symbol": "nA",
          "desc": "电流:纳安(nA)"
        },
        {
          "code": "ohm",
          "type": "电力单位",
          "name": "欧姆",
          "symbol": "Ω",
          "desc": "电阻:欧姆(Ω)"
        },
        {
          "code": "kiloOhm",
          "type": "电力单位",
          "name": "千欧",
          "symbol": "KΩ",
          "desc": "电阻:千欧(KΩ)"
        },
        {
          "code": "millionOhm",
          "type": "电力单位",
          "name": "兆欧",
          "symbol": "MΩ",
          "desc": "电阻:兆欧(MΩ)"
        },
        {
          "code": "electronVolts",
          "type": "电力单位",
          "name": "电子伏",
          "symbol": "eV",
          "desc": "能单位:电子伏(eV)"
        },
        {
          "code": "kWattsHour",
          "type": "电力单位",
          "name": "千瓦·时",
          "symbol": "kW·h",
          "desc": "能单位:千瓦·时(kW·h)"
        }
      ]
    }
``` 