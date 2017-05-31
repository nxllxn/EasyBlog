---
title: net.sf.json的复制问题
tags:
     - java
     - JSON工具
---

### net.sf.json的复制问题


#### net.sf.json 

        Map<String, Object> rootMap = new HashMap<>();
        Map<String, Object> subMapOne = new HashMap<>();
        Map<String, Object> subMapTwo = new HashMap<>();
        

        subMapOne.put("map",subMapTwo);
        rootMap.put("map",subMapOne);
        

        System.out.println(rootMap);        //{map={map={}}}
        

        subMapTwo.put("xxx","xxx");         
        System.out.println(rootMap);        //{map={map={xxx=xxx}}}
        


        JSONObject rootJsonObj = new JSONObject();
        JSONObject subJsonObjOne = new JSONObject();
        JSONObject subJsonObjTwo = new JSONObject();
        

        subJsonObjOne.put("json",subJsonObjTwo);
        rootJsonObj.put("json",subJsonObjOne);
        

        System.out.println(rootJsonObj);        //{"json":{"json":{}}}
        

        subJsonObjTwo.put("xxx","xxx");
        System.out.println(rootJsonObj);        //{"json":{"json":{}}}     What are you 弄啥嘞？
        
        
#### com.alibaba.fastjson

        Map<String, Object> rootMap = new HashMap<>();
        Map<String, Object> subMapOne = new HashMap<>();
        Map<String, Object> subMapTwo = new HashMap<>();
        

        subMapOne.put("map",subMapTwo);
        rootMap.put("map",subMapOne);
        

        System.out.println(rootMap);        //{map={map={}}}
        

        subMapTwo.put("xxx","xxx");         
        System.out.println(rootMap);        //{map={map={xxx=xxx}}}
        

        JSONObject rootJsonObj = new JSONObject();
        JSONObject subJsonObjOne = new JSONObject();
        JSONObject subJsonObjTwo = new JSONObject();
        

        subJsonObjOne.put("json",subJsonObjTwo);
        rootJsonObj.put("json",subJsonObjOne);
        

        System.out.println(rootJsonObj);        //{"json":{"json":{}}}
        

        subJsonObjTwo.put("xxx","xxx");
        System.out.println(rootJsonObj);        //{"json":{"json":{"xxx":"xxx"}}}   ：)
        