---
title: iOS 正则匹配常用方法
date: 2020-07-08 18:06:00
tags: 代码库
---

1. 验证手机号
```
// 验证手机号
+ (BOOL)isValidatePhone:(NSString *)phone{
    NSString *phoneRegex = @"^1([358][0-9]|4[579]|66|7[0135678]|9[89])[0-9]{8}$";
    NSPredicate *phoneTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", phoneRegex];
    return [phoneTest evaluateWithObject:phone];
}
```

2. 邮箱账号有效性判断
```
// 邮箱账号的有效性判断
+ (BOOL)isValidateEmail:(NSString *)email{
    NSString * emailRegex = @"[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4}";
    NSPredicate * emailTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", emailRegex];
    return [emailTest evaluateWithObject:email];
}
```

3. 匹配密码格式(长度6~20位，只能是数字、大小写字母)
```
// 匹配密码格式
+ (BOOL)isValidatePassword:(NSString *)password{
    NSString * passwordRegex = @"[a-zA-Z0-9]{6,20}";
    NSPredicate * passwordTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", passwordRegex];
    return [passwordTest evaluateWithObject:password];
}
```

4. 车牌号码判断
```
// 车牌号码正则表达式
+ (BOOL)isValidateCarID:(NSString *)carID{
    if (carID.length==7) {
        //普通汽车，7位字符，不包含I和O，避免与数字1和0混淆
        NSString *carRegex = @"^[\u4e00-\u9fa5]{1}[a-hj-np-zA-HJ-NP-Z]{1}[a-hj-np-zA-HJ-NP-Z0-9]{4}[a-hj-np-zA-HJ-NP-Z0-9\u4e00-\u9fa5]$";
        NSPredicate *carTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", carRegex];
        return [carTest evaluateWithObject:carID];
    }else if(carID.length==8){
        //新能源车,8位字符，第一位：省份简称（1位汉字），第二位：发牌机关代号（1位字母）;
        //小型车，第三位：只能用字母D或字母F，第四位：字母或者数字，后四位：必须使用数字;([DF][A-HJ-NP-Z0-9][0-9]{4})
        //大型车3-7位：必须使用数字，后一位：只能用字母D或字母F。([0-9]{5}[DF])
        NSString *carRegex = @"^[\u4e00-\u9fa5]{1}[a-hj-np-zA-HJ-NP-Z]{1}([0-9]{5}[d|f|D|F]|[d|f|D|F][a-hj-np-zA-HJ-NP-Z0-9][0-9]{4})$";
        NSPredicate *carTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", carRegex];
        return [carTest evaluateWithObject:carID];
    }
    return NO;
}
```

5. 身份证号判断
```
// 身份证号段正则表达式
+ (BOOL)isValidateIDCard:(NSString *)identityString{
    if (identityString.length != 18) return NO;
    // 正则表达式判断基本 身份证号是否满足格式
    NSString *regex2 = @"^(^[1-9]\\d{7}((0\\d)|(1[0-2]))(([0|1|2]\\d)|3[0-1])\\d{3}$)|(^[1-9]\\d{5}[1-9]\\d{3}((0\\d)|(1[0-2]))(([0|1|2]\\d)|3[0-1])((\\d{4})|\\d{3}[Xx])$)$";
    NSPredicate *identityStringPredicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regex2];
    //如果通过该验证，说明身份证格式正确，但准确性还需计算
    if(![identityStringPredicate evaluateWithObject:identityString]) return NO;
    //** 开始进行校验 *//
    //将前17位加权因子保存在数组里
    NSArray *idCardWiArray = @[@"7", @"9", @"10", @"5", @"8", @"4", @"2", @"1", @"6", @"3", @"7", @"9", @"10", @"5", @"8", @"4", @"2"];
    //这是除以11后，可能产生的11位余数、验证码，也保存成数组
    NSArray *idCardYArray = @[@"1", @"0", @"10", @"9", @"8", @"7", @"6", @"5", @"4", @"3", @"2"];
    //用来保存前17位各自乖以加权因子后的总和
    NSInteger idCardWiSum = 0;
    for(int i = 0;i < 17;i++) {
        NSInteger subStrIndex  = [[identityString substringWithRange:NSMakeRange(i, 1)] integerValue];
        NSInteger idCardWiIndex = [[idCardWiArray objectAtIndex:i] integerValue];
        idCardWiSum += subStrIndex * idCardWiIndex;
    }
    //计算出校验码所在数组的位置
    NSInteger idCardMod=idCardWiSum%11;
    //得到最后一位身份证号码
    NSString *idCardLast= [identityString substringWithRange:NSMakeRange(17, 1)];
    //如果等于2，则说明校验码是10，身份证号码最后一位应该是X
    if(idCardMod==2) {
        if(![idCardLast isEqualToString:@"X"]||[idCardLast isEqualToString:@"x"]) {
            return NO;
        }
    }else{
        //用计算出的验证码与最后一位身份证号码匹配，如果一致，说明通过，否则是无效的身份证号码
        if(![idCardLast isEqualToString: [idCardYArray objectAtIndex:idCardMod]]) {
            return NO;
        }
    }
    return YES;
}
```

6. 随机获取八位字符
```
- (NSString *)obtain8RandomCode {
    NSArray *changeArray = [[NSArray alloc] initWithObjects:@"0",@"1",@"2",@"3",@"4",@"5",@"6",@"7",@"8",@"9",@"A",@"B",@"C",@"D",@"E",@"F",@"G",@"H",@"I",@"J",@"K",@"L",@"M",@"N",@"O",@"P",@"Q",@"R",@"S",@"T",@"U",@"V",@"W",@"X",@"Y",@"Z",@"a",@"b",@"c",@"d",@"e",@"f",@"g",@"h",@"i",@"j",@"k",@"l",@"m",@"n",@"o",@"p",@"q",@"r",@"s",@"t",@"u",@"v",@"w",@"x",@"y",@"z",@"!",@"@",@"#",@"$",@"^",@"&",@"*",@"-",@"+",nil];
    NSArray *specailArray = [[NSArray alloc] initWithObjects:@"!",@"@",@"#",@"$",@"^",@"&",@"*",@"-",@"+", nil];
    NSMutableString *changeString = [[NSMutableString alloc] initWithCapacity:8];
    
    NSInteger specialIndex = arc4random()%7;
    NSInteger specialArrayIndex = arc4random()%([specailArray count] - 1);
    for(int i = 0; i < 8; i++){
        if (i==specialIndex) {
            changeString = (NSMutableString *)[changeString stringByAppendingString:[specailArray objectAtIndex:specialArrayIndex]];
            continue;
        }
        NSInteger index = arc4random()%([changeArray count] - 1);
        changeString = (NSMutableString *)[changeString stringByAppendingString:[changeArray objectAtIndex:index]];
    }
    return changeString;
}
```
