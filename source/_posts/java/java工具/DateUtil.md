---
title: 如何写一个优雅地实现一个Java日期转换工具
tags:
     - java
     - 日期转换工具
     - 日期格式化工具
     - DateTool
---

<p style="height=20;"></p>

### 需求缘起
* 日常编程规程中或多或少需要用到日期转换相关的功能，比如常用格式的日期格式化为字符串或者是字符串转日期对象等，甚至有的时候需要一个兼容日期格式较多的字符串转日期方法，
，继续往下看，我会实现一个基本满足上述功能的简单工具类。
* 为什么要实现为一个工具而不是一个工具类呢。首先声明工具类和工具是有区别的，工具类是指XXXUtil这种命名的类，其内部通常包含一组功能相近或者相似的静态工具函数，注意了这里的静态
工具函数几个字。尔工具的是指以XXXTool这种命名的类，这种类通常是可实例化的内部有一组功能相近或者相似的实例方法。好吧，扯了这么多，其实大概意思就是我会用上一点点面向
对象的设计思路。具体为什么要这样，看完本文你就会大概了解我的初衷的。

### 写在编码之前
任何一份好的代码都必然是经过两次加工，第一次是在脑子里面，我不建议拿到需求就里面开始编码，首先你要把需求整个过一遍，在大脑里面有个大概的思路，此外，程序是写来处理异常
情况的，为什么有“测试驱动编程”，就是我们先明确我们的代码需要处理的全部流程，需要的功能是什么，有哪些异常情况需要处理等等，做到这一步才能说我们理解了需求。
* 基本功能，基本的字符串转日期功能，基本的日期格式化为字符串功能，一个可以兼容多种日期格式的字符串转日期功能。
* 为了方便使用，可以为一些常用的格式转换提供默认实现
* 面向对象编程，提供一组接口，提供这个接口的不同时间，为什么不同？之后再告诉你！
* 单例，我计划将这个工具类设计为一个单例
* 线程安全，每次new一个日期转换工具类太麻烦而且浪费资源，我们会对日期格式化相关的对象做一个“缓存”。什么缓存，直接维护几个全局静态的实例不就好了吗，too young too naive.SimpleDateFormat
不是线程安全的，在并发环境下这种实现就会出问题。那正确的姿势是什么呢？ThreadLocal！
* 单元测试，这里把单元测试列为设计实现的一部分，首先单元测试的重要性我们就不说了，主要是代码写完了，是骡子是马，总的遛一遛看一下对不对。

<!--more-->

### 编码
#### 接口定义
不多说了，直接上代码 
        
        public interface DateTool {
            /**
             * 日志TAG
             */
            String TAG = "DateTool";
        
            /**
             * 默认的日期格式
             */
            String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";  
            
        
            /**
             * 默认月份格式
             */
            String DEFAULT_MONTH_FORMAT = "yyyy-MM";
            
        
            /**
             * 默认年份格式
             */
            String DEFAULT_YEAR_FORMAT = "yyyy";
            
        
            /**
             * 默认的时间格式
             */
            String DEFAULT_TIME_FORMAT = "HH:mm:ss";
            
        
            /**
             * 默认的日期时间格式
             */
            String DEFAULT_DATETIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
            
        
            /**
             * 如果字符串长度为8位，且都是字符串，那么我们认为这是一个yyyyMMdd格式的日期
             */
            Pattern FULL_DATE_PATTERN = Pattern.compile("[0-9]{8}"); //20120101
            
        
            /**
             * 日期分隔符
             */
            String DATE_NOTIFIER_PATTERN_STR = "[年月日\\-/.]+";
            
        
            /**
             * 时间分隔符
             */
            String TIME_NOTIFIER_PATTERN_STR = "[时分秒点:]+";
            
        
            /**
             * 压缩的时间日期格式
             */
            String DATA_TIME_FORMAT = "yyyyMMddHHmmss";
            
        
            /**
             * 压缩的日期格式
             */
            String DATA_FORMAT = "yyyyMMdd";
            
        
            /**
             * 压缩的时间格式
             */
            String TIME_FORMAT = "HHmmss";
            
            Pattern REG_PATTERN_FOR_DATE = Pattern.compile(
                        "((^((1[8-9]\\d{2})|([2-9]\\d{3}))([-/\\._])(10|12|0?[13578])([-/\\._])(3[01]|[12][0-9]|0?[1-9]).*$)" +
                                "|(^((1[8-9]\\d{2})|([2-9]\\d{3}))([-/\\._])(11|0?[469])([-/\\._])(30|[12][0-9]|0?[1-9]).*$)" +
                                "|(^((1[8-9]\\d{2})|([2-9]\\d{3}))([-/\\._])(0?2)([-/\\._])(2[0-8]|1[0-9]|0?[1-9]).*$)" +
                                "|(^([2468][048]00)([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([3579][26]00)([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([1][89][0][48])([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([2-9][0-9][0][48])([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([1][89][2468][048])([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([2-9][0-9][2468][048])([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([1][89][13579][26])([-/\\._])(0?2)([-/\\._])(29).*$)" +
                                "|(^([2-9][0-9][13579][26])([-/\\._])(0?2)([-/\\._])(29).*$))");
        
             /**
             * 使用指定格式化字符转换指定日期字符串
             *
             * @param dateTimeStr 指定日期字符串
             * @return 转换后得到的date
             */
            Date parse(String dateTimeStr) throws Exception ;
        
        
            /**
             * 使用指定格式化字符转换指定日期字符串
             *
             * @param dateString 指定日期字符串
             * @param parseStr   指定格式化字符
             * @return 转换后得到的date
             */
            Date parse(String parseStr, String dateString) throws Exception ;
            
        
            /**
             * 按照指定字符串格式格式化指定日期
             * @param pattern 字符串格式
             * @param date 指定日期
             * @return 格式化后的字符串
             */
            String format(String pattern, Date date) throws Exception ;
            
        
            /**
             * 使用默认日期格式转换指定日期字符换
             *
             * @param dateString 指定日期字符串
             * @return 转换后得到的date
             */
            Date parseDate(String dateString) throws Exception ;
            
        
            /**
             * 使用默认日期格式格式化date
             *
             * @param date 指定date
             * @return 格式化后的字符串
             */
            String formatDate(Date date) throws Exception ;
            
        
            /**
             * 使用默认时间格式转换时间字符串
             *
             * @param dateString 时间字符串
             * @return 转换后得到的date
             */
            Date parseTime(String dateString) throws Exception ;
            
        
            /**
             * 使用默认时间格式格式化date
             *
             * @param date 指定date
             * @return 格式化后的字符串
             */
            String formatTime(Date date) throws Exception ;
            
        
            /**
             * 使用默认格式转换指定日期时间字符串
             *
             * @param dateString 指定日期时间字符串
             * @return 转换后得到的date
             */
            Date parseDateTime(String dateString) throws Exception ;
            
        
            /**
             * 使用默认日期时间格式格式化date
             *
             * @param date 指定date
             * @return 格式化后的字符串
             */
            String formatDateTime(Date date) throws Exception ;
        }
里面有比较详细的JavaDoc，我大概说一下
* 程序开始定义了一些常用的日期格式字符串常量
* 接下来是相关接口一共有9个接口：
    * Date parse(String dateTimeStr);兼容多种日期格式的字符串转日期接口
    * Date parse(String parseStr, String dateString) ;使用指定格式化字符parseStr转换指定日期字符串dateString
    * Date parseDate(String dateString) ; 使用默认日期格式转换指定日期字符换
    * Date parseTime(String dateString) ; 使用默认时间格式转换时间字符串
    * Date parseDateTime(String dateString) ; 使用默认格式转换指定日期时间字符串
    * String format(String pattern, Date date); 按照指定字符串格式格式化指定日期
    * String formatDate(Date date); 使用默认日期格式格式化date
    * String formatTime(Date date) ; 使用默认时间格式格式化date
    * String formatDateTime(Date date); 使用默认日期时间格式格式化date

一个字，清晰！明了！好了接下来就是具体实现了。

#### ThreadLocal实现时间转换格式“缓存”
还是先上代码

        public class BaseDateTool {
            /**
             * ThreadLocal 对象,用于存储当前线程中使用的日期格式化对象
             */
            private static final ThreadLocal convertDataFormatThreadLocal = new ThreadLocal();
        
            /**
             * 获取或者实例化一个指定字符串格式对应的SimpleDateFormat对象
             * @param pattern 格式化字符串
             * @return 格式化字符串对应的SimpleDateFormat格式化对象
             */
            protected SimpleDateFormat getDateFormat(String pattern) throws Exception{
                Map<String,SimpleDateFormat> map = (Map<String, SimpleDateFormat>) convertDataFormatThreadLocal.get();
        
                if(map == null){
                    map = new HashMap<>();
        
                    map.put(pattern,new SimpleDateFormat(pattern));
        
                    convertDataFormatThreadLocal.set(map);
                }
        
                if(!map.containsKey(pattern)){
                    map.put(pattern,new SimpleDateFormat(pattern));
                }
        
                return map.get(pattern);
            }
        }

这是具体实现类的一个基类，既然接口都抽出来了，把这一块功能抽离到一个基类里面好像也不过份吧。
简单解释一下，首先声明了一个全局的静态的final的ThreadLocal对象，不用说了，我们用于日期转换格式化的SimpleDateFormat对象就会放到这里面。
那么问题来了，怎么放呢，实际编码过程中，可能遇到各种类型的日期格式需要进行格式化或者转换。所以我们用一个Map来存储这些对象，key为实际的
字符串格式，值为当前字符串格式对应的日期转换格式化对象，如果Map中已经存在了指定字符串格式的key，那么直接将这个key对应的对象返回，否则
先new一个当前格式的对象，然后插入到Map中并返回。

#### 具体实现
老规矩，先上代码。

        public class DateToolImpl extends BaseDateTool implements DateTool{
             private DateToolImpl(){}
            
            private static class Holder{
                private static final DateTool dateTool = new DateToolImpl();
            }
        
            public static DateTool getSiingleInstance(){
                return Holder.dateTool;
            }
            
            @Override
            public Date parse(String dateTimeStr) throws Exception  {
                if (StringUtils.isEmpty(dateTimeStr)) {
                    return null;
                }
        
                dateTimeStr = dateTimeStr.replaceAll("\\.\\d{3}", "");   //去除掉毫秒部分
        
                String dateStr = null;
                String timeStr = null;
                String[] dateTimeStrParts = dateTimeStr.split("[ \\n]");
                for (String dateTimeStrPart : dateTimeStrParts) {
                    if (REG_PATTERN_FOR_DATE.matcher(dateTimeStrPart).matches()
                            || dateTimeStrPart.contains("年")
                            || dateTimeStrPart.contains("月")
                            || dateTimeStrPart.contains("日")
                            || dateTimeStrPart.contains("-")
                            || dateTimeStrPart.contains("/")
                            || dateTimeStrPart.contains(".")
                            || FULL_DATE_PATTERN.matcher(dateTimeStrPart).matches()
                            || dateTimeStrPart.length() == 4) {
                        dateStr = dateTimeStrPart;
                    } else {
                        timeStr = dateTimeStrPart;
                    }
                }
        
                StringBuilder dateBuilder = new StringBuilder();
                if (!StringUtils.isEmpty(dateStr)) {
                    String[] dateParts = dateStr.split(DATE_NOTIFIER_PATTERN_STR);
                    for (String datePart : dateParts) {
                        if (datePart.length() == 1) {
                            dateBuilder.append("0");
                        }
        
                        dateBuilder.append(datePart);
                    }
        
                    switch (dateBuilder.length()) {
                        case 4:
                            dateBuilder.append("0101");
                            break;
                        case 6:
                            dateBuilder.append("01");
                            break;
                    }
                }
        
        
                StringBuilder timeBuilder = new StringBuilder();
        
                if (!StringUtils.isEmpty(timeStr)) {
                    String[] timeParts = timeStr.split(TIME_NOTIFIER_PATTERN_STR);
                    for (String timePart : timeParts) {
                        if (timePart.length() == 1) {
                            timeBuilder.append("0");
                        }
        
                        timeBuilder.append(timePart);
                    }
        
                    switch (timeBuilder.length()) {
                        case 2:
                            timeBuilder.append("0000");
                            break;
                        case 4:
                            timeBuilder.append("00");
                            break;
                    }
                }
        
                String formattedDateStr = dateBuilder.toString();
                String formattedTimeStr = timeBuilder.toString();
        
                if (StringUtils.isEmpty(formattedDateStr) && StringUtils.isEmpty(formattedTimeStr)) {
                    return null;
                }
        
                if (StringUtils.isEmpty(formattedDateStr)) {
                    return parse(TIME_FORMAT,formattedTimeStr);
                }
        
                if (StringUtils.isEmpty(formattedTimeStr)) {
                    return parse(DATA_FORMAT,formattedDateStr);
                }
        
                return parse(DATA_TIME_FORMAT,formattedDateStr + formattedTimeStr);
            }
        
            @Override
            public Date parse(String parseStr, String dateString) throws Exception {
                if(StringUtils.isEmpty(parseStr) || StringUtils.isEmpty(dateString)){
                    return null;
                }
        
                return getDateFormat(parseStr).parse(dateString);
            }
        
            @Override
            public String format(String pattern, Date date) throws Exception {
                if(StringUtils.isEmpty(pattern) || date == null){
                    return null;
                }
        
                return getDateFormat(pattern).format(date);
            }
        
            @Override
            public Date parseDate(String dateString) throws Exception {
                return parse(DEFAULT_DATE_FORMAT, dateString);
            }
        
            @Override
            public String formatDate(Date date) throws Exception  {
                return format(DEFAULT_DATE_FORMAT, date);
            }
        
            @Override
            public Date parseTime(String dateString) throws Exception {
                return parse(DEFAULT_TIME_FORMAT, dateString);
            }
        
            @Override
            public String formatTime(Date date) throws Exception  {
                return format(DEFAULT_TIME_FORMAT, date);
            }
        
            @Override
            public Date parseDateTime(String dateString) throws Exception {
                return parse(DEFAULT_DATETIME_FORMAT, dateString);
            }
        
            @Override
            public String formatDateTime(Date date) throws Exception  {
                return format(DEFAULT_DATETIME_FORMAT, date);
            }
        }

具体实现细节我就不再多讲了，上面的每个实现都不是很复杂。

#### 测试

        DateTool dateTool = DateToolImpl.getSingleInstance();
        @Test
        @SuppressWarnings("unchecked")
        public void testIntelligentParse(){
            assertEquals("Thu Feb 02 00:00:00 CST 2012",dateTool.parse("20120202").toString());
            assertEquals("Thu Feb 02 00:00:00 CST 2012",dateTool.parse("2012-02-02").toString());
            assertEquals("Thu Feb 02 00:00:00 CST 2012",dateTool.parse("2012-02-2").toString());
            assertEquals("Thu Feb 02 00:00:00 CST 2012",dateTool.parse("2012年2月2日").toString());
    
            assertEquals("Thu Feb 02 01:01:01 CST 2012",dateTool.parse("2012-02-02 01:01:01").toString());
            assertEquals("Thu Feb 02 00:00:00 CST 2012",dateTool.parse("2012/02/02").toString());
            assertEquals("Thu Feb 02 00:00:00 CST 2012",dateTool.parse("2012-02-02").toString());
            assertEquals("Thu Jan 01 20:12:02 CST 1970",dateTool.parse("201202").toString());
    
            assertEquals("Sun Jan 01 00:00:00 CST 2012",dateTool.parse("2012").toString());
            assertEquals("Sun Jan 01 00:00:00 CST 2012",dateTool.parse("2012年").toString());
            assertEquals("Wed Feb 01 00:00:00 CST 2012",dateTool.parse("2012年02月").toString());
            assertEquals("Wed Feb 01 00:00:00 CST 2012",dateTool.parse("2012.02.01").toString());
    
            assertEquals("Wed Feb 01 00:00:00 CST 2012",dateTool.parse("2012.02").toString());
            assertEquals("Thu Jan 01 11:11:11 CST 1970",dateTool.parse("11:11:11").toString());
            assertEquals("Thu Jan 01 01:01:01 CST 1970",dateTool.parse("1:1:1").toString());
            assertEquals("Thu Jan 01 01:01:01 CST 1970",dateTool.parse("01:01:01").toString());
    
            assertEquals("Thu Jan 01 11:11:00 CST 1970",dateTool.parse("11:11").toString());
            assertEquals("Thu Jan 01 11:00:00 CST 1970",dateTool.parse("11").toString());
        }
        
        结果：上面的单元测试都可以通过
        
        //测试多线程
        public static void main(String[] args) {
            DateTool dateTool = DateToolImpl.getSingleInstance();
            for (int index = 0;index < 10;index ++){
                new Thread(() -> {
                    String currentThreadName = Thread.currentThread().getName();
    
                    System.out.println(currentThreadName + " " + dateTool.parse("20120202"));
                    System.out.println(currentThreadName + " " + dateTool.parse("11:11:11"));
                    System.out.println(currentThreadName + " " + dateTool.parse("2012年02月02日 11:11:11"));
                }).start();
            }
        }
        
        结果：
        Thread-7 Thu Feb 02 00:00:00 CST 2012
        Thread-8 Thu Feb 02 00:00:00 CST 2012
        Thread-2 Thu Feb 02 00:00:00 CST 2012
        Thread-7 Thu Jan 01 11:11:11 CST 1970
        Thread-2 Thu Jan 01 11:11:11 CST 1970
        Thread-3 Thu Feb 02 00:00:00 CST 2012
        Thread-5 Thu Feb 02 00:00:00 CST 2012
        Thread-3 Thu Jan 01 11:11:11 CST 1970
        Thread-5 Thu Jan 01 11:11:11 CST 1970
        Thread-6 Thu Feb 02 00:00:00 CST 2012
        Thread-0 Thu Feb 02 00:00:00 CST 2012
        Thread-3 Thu Feb 02 11:11:11 CST 2012
        Thread-2 Thu Feb 02 11:11:11 CST 2012
        Thread-9 Thu Feb 02 00:00:00 CST 2012
        Thread-4 Thu Feb 02 00:00:00 CST 2012
        Thread-1 Thu Feb 02 00:00:00 CST 2012
        Thread-0 Thu Jan 01 11:11:11 CST 1970
        Thread-7 Thu Feb 02 11:11:11 CST 2012
        Thread-1 Thu Jan 01 11:11:11 CST 1970
        Thread-8 Thu Jan 01 11:11:11 CST 1970
        Thread-0 Thu Feb 02 11:11:11 CST 2012
        Thread-1 Thu Feb 02 11:11:11 CST 2012
        Thread-8 Thu Feb 02 11:11:11 CST 2012
        Thread-6 Thu Jan 01 11:11:11 CST 1970
        Thread-5 Thu Feb 02 11:11:11 CST 2012
        Thread-4 Thu Jan 01 11:11:11 CST 1970
        Thread-6 Thu Feb 02 11:11:11 CST 2012
        Thread-9 Thu Jan 01 11:11:11 CST 1970
        Thread-4 Thu Feb 02 11:11:11 CST 2012
        Thread-9 Thu Feb 02 11:11:11 CST 2012

#### 结语
只是一个小demo，代码写的不好，有什么问题恳请批评指正，感激涕零！此外，请轻喷！