---
title: 如何优雅的操作一个Json
tags:
     - java
     - JSON工具
     - JSONTool
---

### 需求缘起
JSON（JavaScript Object Notation）,是一种被广泛使用的后端数据处理以及前后端数据交互格式，本文将简要讨论一下如何中规中矩的操作一个JsonObject，从中读取数据，最后我也会简单介绍一下如何优雅地构建一个Json。

#### 写在编码之前
需求定义：
1）从一个Json对象中读取Json属性，提供一个基本的读取函数get（返回值为Object），以及一些常用的类型如String，Double，Integer，Long，Boolean，JSONArray，JSONObject的读取函数
2）为上述所有函数提供一个包含默认值的重载函数，当用户调用这个有默认值的函数时，编码更加清晰，程序的行为会更加明了
3）虽然提供默认值程序会更加明了，但是上述两种解决方案还是可能导致大量空指针异常，为了对这个情况进行控制我们选择使用Optional（不了解的可参考guava类库以及Java8 Optional）来为上述函数统一提供一个Optional版本的重载（其实并不能叫重载，叫实现好一点，这里是为了凸显这三种实现的共性）。
4）跨越多级Json取出指定属性
5）原始的Json构造模式中代码不是很美观，而且如果你使用net.sf.json类库的话构造Json还可能出问题（之后会提到），所以本文最后会利用构造器模式来相对优雅地进行Json的组装。
6）问题5中的一个问题，先声明，此问题只有net.sf.json中会出现，fastjson类库中没有上述问题。其它类库中TZ没有进行过研究。

<!--more-->

### 编码
#### 首先是一个单例

        public class JSONTool {
            private JSONTool() {
            }
        
            private static class Holder {
                private static JSONTool jsonTool = new JSONTool();
            }
        
            /**
             * 单例，才疏学浅，只会这么一个设计模式，哎
             *
             * @return JsonTool对象
             */
            public static JSONTool getSingleInstance() {
                return Holder.jsonTool;
            }
        }

#### 中规中矩的数据读取方式（包括带默认值的）
        
        /**
         * 判断当前Json对象中指定Key对应的json属性是否存在
         *
         * @param dataJsonObj 当前jsonObject
         * @param key         指定json属性对应的key
         * @return 存在当前key返回true，否则返回false
         */
        public Boolean isAttributeExists(JSONObject dataJsonObj, String key) {
            return !(dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) && dataJsonObj.containsKey(key);
        }
    
        /**
         * 从当前json对象中取出指定key对应的布尔属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 返回指定key对应的布尔属性，如果指定key不存在或者不是一个有效的bool值，返回null
         */
        public Boolean getBoolean(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) {
                return null;
            }
    
            String strValue = getString(dataJsonObj, key);
            if (StringUtils.isEmpty(strValue)) {
                return null;
            }
    
            if (!REGUtil.REG_PATTERN_FOR_BOOLEAN.matcher(strValue).matches()) {
                return null;
            }
            
            return Boolean.valueOf(strValue);
        }
        
    
        /**
         * 从当前json对象中取出指定key对应的布尔属性
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 返回指定key对应的布尔属性，如果指定key不存在或者不是一个有效的bool值，返回defaultValue
         */
        public Boolean getBoolean(JSONObject dataJsonObj, String key, Boolean defaultValue) {
            Boolean actualValue = getBoolean(dataJsonObj, key);
            
            return actualValue == null ? defaultValue : actualValue;
        }

        /**
         * 从当前json对象中取出指定键对应的双精度浮点数属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 如果指定key不存在或者不是一个有效的数值属性，返回null
         */
        public Double getDouble(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) {
                return null;
            }
    
            String strValue = getString(dataJsonObj, key);
            if (StringUtils.isEmpty(strValue)) {
                return null;
            }
    
            if (!REGUtil.REG_PATTERN_FOR_NUMBER.matcher(strValue).matches()) {
                return null;
            }
    
            return Double.valueOf(strValue.replaceAll(REGUtil.INVALID_FLOAT_CHAR_REG, ""));
        }
    
        /**
         * 从指定Json对象中取出指定Key对应的数值属性值
         *
         * @param dataJsonObj  指定Json对象
         * @param key          指定Key
         * @param defaultValue 默认值
         * @return 如果指定key对应的属性为有效的数值属性，那么返回此值，否则返回默认值defaultValue
         */
        public Double getDouble(JSONObject dataJsonObj, String key, Double defaultValue) {
            Double actualValue = getDouble(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue;
        }
    
        /**
         * 从当前json对象中取出指定键对应的金额属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 如果指定key不存在或者不是一个有效的金额属性，返回null
         */
        public Double getAmount(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) {
                return null;
            }
    
            String strValue = getString(dataJsonObj, key);
            if (StringUtils.isEmpty(strValue)) {
                return null;
            }
            
            //替换掉非数值字符
            strValue = strValue.replaceAll(REGUtil.INVALID_FLOAT_CHAR_REG, "");
    
            if (!REGUtil.REG_PATTERN_FOR_NUMBER.matcher(strValue).matches()) {
                return null;
            }
    
            return Double.valueOf(strValue);
        }
        
        /**
         * 从指定Json对象中取出指定Key对应的金额属性值
         *
         * @param dataJsonObj  指定Json对象
         * @param key          指定Key
         * @param defaultValue 默认值
         * @return 如果指定key对应的属性为有效的金额属性，那么返回此值，否则返回默认值defaultValue
         */
        public Double getAmount(JSONObject dataJsonObj, String key, Double defaultValue) {
            Double actualValue = getAmount(dataJsonObj, key);
            
            return actualValue == null ? defaultValue : actualValue;
        }
    
        /**
         * 从当前对象中取出指定键对应的单精度浮点数属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 指定键对应的单精度浮点数属性，如果指定key不存在或者不是一个有效的数值属性，返回null
         */
        public Float getFloat(JSONObject dataJsonObj, String key) {
            Double actualValue = getDouble(dataJsonObj, key);
    
            return actualValue == null ? null : actualValue.floatValue();
        }
        
    
        /**
         * 从当前对象中取出指定键对应的单精度浮点数属性
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定键对应的单精度浮点数属性，如果指定key不存在或者不是一个有效的数值属性，返回默认值
         */
        public Float getFloat(JSONObject dataJsonObj, String key, Float defaultValue) {
            Double actualValue = getDouble(dataJsonObj, key);
            
            return actualValue == null ? defaultValue : actualValue.floatValue();
        }
        
    
        /**
         * 从当前对象中取出指定键对应的属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 指定键对应的属性，如果不存在或发生异常返回null
         */
        public Object get(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty()) {
                return null;
            }
    
            if (!dataJsonObj.containsKey(key)) {
                return null;
            }
            
            return dataJsonObj.get(key);
        }
        
    
        /**
         * 从当前对象中取出指定键对应的属性
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定键对应的属性，如果不存在返回默认值
         */
        public Object get(JSONObject dataJsonObj, String key, Object defaultValue) {
            Object actualValue = get(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue;
        }
        
    
        /**
         * 从当前对象中取出指定键对应的整数属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 指定键对应的整数属性，如果指定key对应的属性不存在或者指定属性不是一个有效的数值属性那么返回null
         */
        public Integer getInteger(JSONObject dataJsonObj, String key) {
            Double actualValue = getDouble(dataJsonObj, key);
    
            return actualValue == null ? null : actualValue.intValue();
        }
        
    
        /**
         * 从当前对象中取出指定键对应的整数属性
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定键对应的整数属性，如果指定key对应的属性不存在或者指定属性不是一个有效的数值属性那么返回默认值
         */
        public Integer getInteger(JSONObject dataJsonObj, String key, Integer defaultValue) {
            Double actualValue = getDouble(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue.intValue();
        }
        
    
        /**
         * 从当前对象中取出指定键对应的长整型整数属性
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 指定键对应的整数属性，如果指定key对应的属性不存在或者不是一个有效的数值属性那么返回null
         */
        public Long getLong(JSONObject dataJsonObj, String key) {
            Double actualValue = getDouble(dataJsonObj, key);
    
            return actualValue == null ? null : actualValue.longValue();
        }
        
    
        /**
         * 从当前对象中取出指定键对应的长整型整数属性
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定键对应的整数属性，如果指定key对应的属性不存在或者不是一个有效的数值属性那么返回默认值
         */
        public Long getLong(JSONObject dataJsonObj, String key, Long defaultValue) {
            Double actualValue = getDouble(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue.longValue();
        }
        
    
        /**
         * 从当前对象中取出指定键对应的json数组
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 指定key对应的json数组，如果指定对应的属性不存在或者不是json数组返回null
         */
        public JSONArray getJsonArray(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) {
                return null;
            }
    
            Object obj = get(dataJsonObj, key);
            if (obj == null || !(obj instanceof JSONArray)) {
                return null;
            }
            
            return (JSONArray) obj;
        }
        
    
        /**
         * 从当前对象中取出指定键对应的json数组
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定key对应的json数组，如果指定对应的属性不存在或者不是Json数组返回默认值
         */
        public JSONArray getJsonArray(JSONObject dataJsonObj, String key, JSONArray defaultValue) {
            JSONArray actualValue = getJsonArray(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue;
        }
        
    
        /**
         * 从当前json对象中取出指定键对应的json对象
         *
         * @param dataJsonObj 当前json对象
         * @param key         指定key
         * @return 指定key对应的json对象属性，如果指定对应的属性不存在或者不是json对象返回null
         */
        public JSONObject getJsonObject(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) {
                return null;
            }
    
            Object obj = get(dataJsonObj, key);
            if (obj == null || !(obj instanceof JSONObject)) {
                return null;
            }
    
            return (JSONObject) obj;
        }
        
    
        /**
         * 从当前json对象中取出指定键对应的json对象
         *
         * @param dataJsonObj  当前json对象
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定key对应的json对象属性，如果指定对应的属性不存在或者不是json对象返回默认值
         */
        public JSONObject getJsonObject(JSONObject dataJsonObj, String key, JSONObject defaultValue) {
            JSONObject actualValue = getJsonObject(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue;
        }
        
    
        /**
         * 从当前json对象中取出指定键对应的字符串
         *
         * @param dataJsonObj 当前jsonObject
         * @param key         指定key
         * @return 指定键对应的字符串，如果不存在返回null
         */
        public String getString(JSONObject dataJsonObj, String key) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)) {
                return null;
            }
    
            if (!dataJsonObj.containsKey(key)) {
                return null;
            }
            
            Object obj = get(dataJsonObj, key);
    
            if (obj == null || obj instanceof JSON) {
                return null;
            }
    
            return String.valueOf(obj);
        }
        
    
        /**
         * 从当前json对象中取出指定键对应的字符串
         *
         * @param dataJsonObj  当前jsonObject
         * @param key          指定key
         * @param defaultValue 默认值
         * @return 指定键对应的字符串，如果不存在或者指定key对应的属性是一个Json对象或者是Json数组返回默认值
         */
        public String getString(JSONObject dataJsonObj, String key, String defaultValue) {
            String actualValue = getString(dataJsonObj, key);
    
            return actualValue == null ? defaultValue : actualValue;
        }

#### Optional方式的数据读取
        
        public Optional<Object> opt(JSONObject dataJsonObj,String key){
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)){
                return Optional.empty();
            }
    
            BiFunction<JSONObject,String,Optional<Object>> optFunction =
                    (x,y) -> Stream.of(x.entrySet())
                            .flatMap(Collection::stream)
                            .filter(z -> ((Map.Entry)z).getKey().equals(y))
                            .map(z -> ((Map.Entry)z).getValue())
                            .findAny();
    
            return optFunction.apply(dataJsonObj,key);
        }
        
    
        public Optional<String> optString(JSONObject dataJsonObj,String key){
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)){
                return Optional.empty();
            }
    
            Optional<Object> optObj = opt(dataJsonObj,key);
            
            if (!optObj.isPresent()){
                return Optional.empty();
            }
    
            Object obj = optObj.get();
            if (obj instanceof JSON) {
                return Optional.empty();
            }
    
            return Optional.of(String.valueOf(obj));
        }
        
    
        /**
         * 返回Optional类型的Double属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的Double属性
         */
        public Optional<Double> optDouble(JSONObject dataJsonObj,String key){
            Optional<String> optStr = optString(dataJsonObj,key);
            
            if (!optStr.isPresent()){
                return Optional.empty();
            }
            
            if (!REGUtil.REG_PATTERN_FOR_NUMBER.matcher(optStr.get()).matches()) {
                return Optional.empty();
            }
    
            return Optional.of(Double.valueOf(optStr.get().replaceAll(REGUtil.INVALID_FLOAT_CHAR_REG, "")));
        }
        
    
        /**
         * 返回Optional类型的Float属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的Float属性
         */
        public Optional<Float> optFloat(JSONObject dataJsonObj,String key){
            Optional<Double> optDouble = optDouble(dataJsonObj,key);
            
            return optDouble.map(Double::floatValue);
        }
        
    
        /**
         * 返回Optional类型的Integer属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的Integer属性
         */
        public Optional<Integer> optInteger(JSONObject dataJsonObj,String key){
            Optional<Double> optDouble = optDouble(dataJsonObj,key);
            
            return optDouble.map(Double::intValue);
        }
        
        /**
         * 返回Optional类型的Long属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的Long属性
         */
        public Optional<Long> optLong(JSONObject dataJsonObj,String key){
            Optional<Double> optDouble = optDouble(dataJsonObj,key);
            
            return optDouble.map(Double::longValue);
        }
    
        /**
         * 返回Optional类型的Boolean属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的Boolean属性
         */
        public Optional<Boolean> optBoolean(JSONObject dataJsonObj,String key) {
            Optional<String> optStr = optString(dataJsonObj,key);
    
            if (!optStr.isPresent()){
                return Optional.empty();
            }
    
            if (StringUtils.isEmpty(optStr.get())) {
                return Optional.empty();
            }
    
            if (!REGUtil.REG_PATTERN_FOR_BOOLEAN.matcher(optStr.get()).matches()) {
                return Optional.empty();
            }
    
            return Optional.of(Boolean.valueOf(optStr.get()));
        }
        
        /**
         * 返回Optional类型的JsonObject属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的JsonObject属性
         */
        public Optional<JSONObject> optJsonObj(JSONObject dataJsonObj,String key){
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)){
                return Optional.empty();
            }
    
            Optional<Object> optObj = opt(dataJsonObj,key);
    
            if (!optObj.isPresent()){
                return Optional.empty();
            }
            
            Object obj = optObj.get();
            if (!(obj instanceof JSONObject)){
                return Optional.empty();
            }
    
            return Optional.of((JSONObject)obj);
        }
    
        /**
         * 返回Optional类型的JsonArray属性
         * @param dataJsonObj 指定Json对象
         * @param key 指定key
         * @return 指定key对应的JsonArray属性
         */
        public Optional<JSONArray> optJsonObjs(JSONObject dataJsonObj,String key){
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(key)){
                return Optional.empty();
            }
            
            Optional<Object> optObj = opt(dataJsonObj,key);
    
            if (!optObj.isPresent()){
                return Optional.empty();
            }
    
            Object obj = optObj.get();
            if (!(obj instanceof JSONArray)){
                return Optional.empty();
            }
    
            return Optional.of((JSONArray)obj);
        }
        
#### 跨越多级取出指定属性

        private static final String FIELD_EXPRESSION_SEPARATOR = ">";
        /**
         * 从指定Json对象中取出指定表达式对应的属性
         *
         * @param dataJsonObj      指定Json对象
         * @param operandExpression 指定属性表达式
         * @return 指定字段对应的属性
         */
        public static Object metaGet(JSONObject dataJsonObj, String operandExpression) {
            if (dataJsonObj == null || dataJsonObj.isEmpty() || StringUtils.isBlank(operandExpression)){
                return null;
            }
    
            String[] operandExpressionParts = operandExpression.split(FIELD_EXPRESSION_SEPARATOR);
            Object obj = dataJsonObj;
            for (String operandExpressionPart : operandExpressionParts) {
                if (obj == null) {
                    break;
                }
                
                if (obj instanceof JSONObject) {
                    obj = ((JSONObject) obj).get(operandExpressionPart);
    
                    continue;
                }
    
                if (obj instanceof JSONArray) {
                    if (((JSONArray) obj).size() == 0) {
                        return null;
                    }
    
                    obj = ((JSONArray) obj).get(0);
    
                    if (obj instanceof JSONObject) {
                        obj = ((JSONObject) obj).get(operandExpressionPart);
                    }
                    
                    continue;
                }
    
                // 如果已经取到基本类型这一层了,但是表达式每一个小块儿还没用完,那么直接返回空,这是一个无效操作
                return null;
            }
            
            return obj;
        }

#### 构造Json

        /**
         * 构造器
         */
        public static class Builder {
            /**
             * 待构造Json对象
             */
            private Map<String, Object> currentMap;
    
            /**
             * 构造函数
             */
            public Builder() {
                this.currentMap = new HashMap<>();
            }
            
            /**
             * 构造指定key-value-pair
             *
             * @param key   指定key
             * @param value 指定值
             * @return 当前构造器对象
             */
            public Builder with(String key, Object value) {
                this.currentMap.put(key, value);
    
                return this;
            }
    
            /**
             * 构建Json对象
             *
             * @return json对象
             */
            public JSONObject build() {
                return JSONObject.fromObject(this.currentMap);
            }
        }
        
        //使用方式
        new JSONTool.Builder()
                .with("key_one","valueOne")
                .with("key_two",1)
                .with("key_three",true)
                .with("key_four",new JSONTool.Builder().with("sub_key_one","subValueOne").build())
                .build();

### net.sf.json的深复制问题
大家应该也注意到上面的Builder中我并没有直接使用JsonObject作为属性，而是使用Map。为什么呢，我们看一下下面的例子，大家就明白了。

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
        