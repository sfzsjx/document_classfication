# 1. Swagger 配置添加
```java
 @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .directModelSubstitute(LocalTime.class,String.class)
                .directModelSubstitute(LocalDateTime.class,String.class)
                .directModelSubstitute(ZonedDateTime.class,String.class)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.leayun.cn.device.controller"))
                .paths(PathSelectors.any())
                .build();
    }
```



# 2. 传递参数前添加注解

>  @DateTimeFormat(patten="HH:mm:ss")

```java
  @GetMapping(value = "/test")
    @ApiOperation(value = "判断车辆是否进入电子围栏")
    public CommonResult<Object> test( @DateTimeFormat(pattern = "HH:mm:ss") @RequestParam(value = "siteCode") LocalTime siteCode){

        System.out.println(siteCode.getHour());
        return CommonResult.success();


    }
```

