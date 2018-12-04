> 示例源码https://github.com/zhuxiujia/GoMybatis/tree/master/example

实际使用Example_test.go
```
import (
	_ "github.com/go-sql-driver/mysql"
	"testing"
	"time"
	"os"
	"fmt"
	"io/ioutil"
	"github.com/zhuxiujia/GoMybatis"
)

//定义mapper文件的接口和结构体
//支持包含基本类型的结构体和基本类型(int,string,time.Time,float...且需要指定参数名称`mapperParams:"name"以逗号隔开，且位置要和实际参数相同)
//参数中除了session指针外，为指针类型的皆为返回数据
// 函数return必须为error 为返回错误信息
type ExampleActivityMapperImpl struct {
	SelectByIds       func(ids []string, result *[]Activity) error `mapperParams:"ids"`
	SelectAll         func(result *[]Activity) error
	SelectByCondition func(name string, startTime time.Time, endTime time.Time, page int, size int, result *[]Activity) error `mapperParams:"name,startTime,endTime,page,size"`
	UpdateById        func(session *GoMybatis.Session, arg Activity, result *int64) error                                     //只要参数中包含有*GoMybatis.Session的类型，框架默认使用传入的session对象，用于自定义事务
	Insert            func(arg Activity, result *int64) error
	CountByCondition  func(name string, startTime time.Time, endTime time.Time, result *int) error `mapperParams:"name,startTime,endTime"`
	DeleteById        func(id string, result *int64) error                                         `mapperParams:"id"`
}


//初始化mapper文件和结构体
func InitMapperByLocalSession() ExampleActivityMapperImpl {
	var err error
	//mysql链接格式为         用户名:密码@(数据库链接地址:端口)/数据库名称   例如root:123456@(***.mysql.rds.aliyuncs.com:3306)/test
	engine, err := GoMybatis.Open("mysql", MysqlUri) //此处请按格式填写你的mysql链接，这里用*号代替
	if err != nil {
		panic(err.Error())
	}
	//读取mapper xml文件
	file, err := os.Open("Example_ActivityMapper.xml")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	bytes, _ := ioutil.ReadAll(file)
	var exampleActivityMapperImpl ExampleActivityMapperImpl
	//设置对应的mapper xml文件
	GoMybatis.UseProxyMapperByEngine(&exampleActivityMapperImpl, bytes, engine)
	return exampleActivityMapperImpl
}
```

### <a name="1">基本使用</a>

```
//本地GoMybatis使用例子
func Test_main(t *testing.T) {
	//初始化mapper文件
	var exampleActivityMapperImpl = InitMapperByLocalSession()
	//使用mapper
	var result []Activity
	var err = exampleActivityMapperImpl.SelectByCondition("", time.Time{}, time.Time{}, 0, 2000, &result)
	if err != nil {
		panic(err)
	}
	fmt.Println("result=", result)
}
```