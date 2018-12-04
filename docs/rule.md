# 限制和规则

# 定义mapper
```
type ActivityMapperImpl struct {
//定义mapper文件的接口和结构体，也可以只定义结构体就行
//mapper.go文件 函数参数（自定义结构体参数（属性必须大写），为指针类型的返回数据,*GoMybatis.Session作为该sql执行的session） error 为返回错误
	SelectAll         func(result *[]Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity, result *int64) error //只要参数中包含有*GoMybatis.Session的类型，框架默认使用传入的session对象，用于自定义事务
	Insert            func(arg Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error                            `mapperParams:"name,startTime,endTime"`
}
```

> 包含一系列func的struct 为mapper定义文件,其中 func 返回值必须为error。当最后一个参数为指针时，表示是返回值类型其他默认为参数，多个参数请定义tag注解  `mapperParams:"***"`,逗号隔开，参数为你的xml中sql出现的‘#{***}’字段

# 定义Mapper xml
xml文件案例:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper>
    <!-- SelectByCondition func(arg SelectByConditionArg, result *[]model.Activity) error -->
    <!-- 后台查询产品 -->
    <select id="SelectByCondition" >
        select
        <trim prefix="" suffix="" suffixOverrides=",">
            <if test="Name != ''">name,</if>
        </trim>
        from biz_activity where delete_flag=1
        <if test="Name != ''">
            and name like concat('%',#{Name},'%')
        </if>
        <if test="StartTime != 0">
            and create_time >= #{StartTime}
        </if>
        <if test="EndTime != 0">
            and create_time &lt;= #{EndTime}
        </if>
        order by create_time desc
        <if test="Page != 0 and Size != 0">limit #{Page}, #{Size}</if>
    </select>
</mapper>
```

> 这里可以看到 #{EndTime} 和 mapperParams定义的endTime不一样，框架默认不区分 参数首字母大小写。if表达式判断参数是否0值，注意 time.Time的零值为0，数值的零值为0，string的零值为''