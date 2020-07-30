---
title: iOS 时间日期总结
date: 2020-04-22 15:12:00
tags: 代码库
---

1. 获取时间戳 
* 单位秒，保留六位有效数字，格式如：`1574068247.545103`
```
NSDate *datenow = [NSDate date];
NSString *timeSp = [NSString stringWithFormat:@"%f", (double)[datenow timeIntervalSince1970]];
```
* 单位秒，整数，格式如：`1574068265`
```
NSDate *datenow = [NSDate date];
NSString *timeSp = [NSString stringWithFormat:@"%ld", (long)[datenow timeIntervalSince1970]];
```
* 单位毫秒，整数，不精确，后面直接补三个`0`，格式如：`1574068602000`
```
NSDate *datenow = [NSDate date];
NSString *timeSp = [NSString stringWithFormat:@"%ld", (long)[datenow timeIntervalSince1970]*1000];
```
* 单位毫秒，整数，精确，格式如：`1574070082387`
```
// 获取当前时间0秒后的时间
NSDate *date = [NSDate dateWithTimeIntervalSinceNow:0];
// *1000 是精确到毫秒，不乘就是精确到秒
NSTimeInterval time = [date timeIntervalSince1970]*1000;
NSString *timeStr = [NSString stringWithFormat:@"%.0f", time];
```

2. 时间戳转日期
```
// 传入的时间戳timeStr如果是精确到毫秒的记得要/1000
NSTimeInterval timeInterval = [timeStr doubleValue]/1000;
NSDate *detailDate = [NSDate dateWithTimeIntervalSince1970:timeInterval];
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
// 实例化一个NSDateFormatter对象，设定时间格式，这里可以设置成自己需要的格式
[dateFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss SS"];
NSString *dateStr = [dateFormatter stringFromDate:detailDate];
```

3. 两个日期比较
```
//1.将这两个时间戳转换成日期
NSDate *date1 = [NSDate dateWithTimeIntervalSince1970:1451047216];
NSDate *date2 = [NSDate dateWithTimeIntervalSince1970:1451847216];
//2.开始比较
// 比较date1是不是比date2早——>会返回一个比较早的日期
NSDate *date3 = [date1 earlierDate:date2];
NSLog(@"比较早的日期：%@",date3);
//比较两个日期谁比谁晚
NSDate *date4 = [date1 laterDate:date2];
NSLog(@"比较晚的日期：%@",date4);
//  比较两个日期 是不是相同 ——>返回值BOOL类型
BOOL result = [date1 isEqualToDate:date2];
NSLog(@"%d",result);
```
* 可以解决跨年、跨月、平闰年时间处理问题
```
// 100天后
NSDate *date = [NSDate dateWithTimeIntervalSinceNow:60 * 60 * 24 * 100];
NSDate *nowDate = [NSDate date];
// 日期升序
if ([nowDate compare:date] == NSOrderedAscending) {
    NSLog(@"如果打印，nowDate比ndate时间早，如nowDate=2019-11-18， ndate=2020-02-26");
}
```

4. 日历组件`NSCalendar`
```
NSDate *nowDate = [NSDate date];
NSCalendar *calendar = [NSCalendar currentCalendar];
// 初始化日历组件，可以选择需要的组件
NSDateComponents *comps = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay|NSCalendarUnitWeekday|NSCalendarUnitWeekdayOrdinal|NSCalendarUnitWeekOfMonth|NSCalendarUnitWeekOfYear|NSCalendarUnitYearForWeekOfYear fromDate:nowDate];
// 得到：今天是星期几，返回日期的工作日索引（1 =星期日，2 =星期一，…，7 =星期六）
NSInteger weekDay = [comps weekday];
// 得到：今天是几号
NSInteger day = [comps day];
// 得到：一年中的第几周
NSInteger weekOfYear = [comps weekOfYear];
```

5. 常见`NSDateFormatter`格式
可以使用以下`dateFormatter`符号单独格式化，拿到需要的数据进行处理

|符号|说明|
|---|---|
|`y/yyy/yyyy/Y/YYY/YYYY/u/uu/uuu/uuuu/U/UUU/UUUU`|完整的年份|
|`yy/YY/UU`|2个数字的年份|
|`M/MM/L/LL`|1~12 第几月|
|`MMM/LLL`|Jan/Feb/Mar/Apr/May/Jun/Jul/Aug/Sep/Oct/Nov/Dec 月份简写|
|`MMMM/LLLL`|January/February/March/April/May/June/July/August/September/October/November/December 月份全称|
|`d`|1~31 (月份的第几天，带0)|
|`D`|1~366 (年份的第几天，带0)|
|`e/c/cc`|1~7 (一周的第几天，周日为1，带0)|
|`E~EEE/eee/ccc`|Sun/Mon/Tue/Wed/Thu/Fri/Sat (星期简写)|
|`EEEE/eeee/cccc`|Sunday/Monday/Tuesday/Wednesday/Thursday/Friday/Saturday (星期全拼)|
|`H`|0~23 带0的时，24小时制|
|`h`|1~12 带0的时，12小时制|
|`k`|1~24 一天中的小时数，带0的时，24小时制|
|`K`|0~11 带0的时，12小时制|
|`m`|0~59 分钟|
|`s`|0~59 秒数|
|`SSS`|毫秒|
|`a`|AM/PM (上午/下午)|
|`A`|0~86399999 (一天的第几微秒)|
|`F`|1~5 每月的第几周|
|`w`|1~53 一年的第几周，一周的开始为周日，第一周从去年的最后一个周日起算|
|`W`|1~5 一个月的第几周，一周的开始为周日|
|`q/qq/Q/QQ`|1~4 第几季度|
|`qqq/QQQ`|Q1/Q2/Q3/Q4 季度简写|
|`qqqq/QQQQ`|1st quarter/2nd quarter/3rd quarter/4th quarter 季度全拼|
|`z~zzz`|指定GMT时区的缩写，GMT+8|
|`zzzz/vvvv`|指定GMT时区的名称，China Standard Time|
|`Z~ZZZ`|指定GMT时区的缩写，+0800|
|`ZZZZ`|指定GMT时区的缩写，GMT+08:00|
|`v/VVVV`|指定GMT时区的名称，China mainland Time|
|`VV`|指定GMT时区的名称，Asia/Shanghai|
|`VVV`|指定GMT时区的名称，Shanghai|
