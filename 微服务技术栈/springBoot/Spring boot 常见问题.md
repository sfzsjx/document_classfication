### 1、 Spring Boot 测试时注入mapper报空指针，注入失败

解决办法：在注入时加入注解@RunWith(SpringJUnit4ClassRunner.class)

案例：

```java
package com.leayun.phecda.rabbitmq;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.leayun.phecda.bean.dao.sheetmetal.TaskRecord;
import com.leayun.phecda.mapper.sheetmetal.TaskRecordMapper;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import javax.annotation.Resource;

@SpringBootTest
@RunWith(SpringJUnit4ClassRunner.class)
public class TaskRecordTest {
    @Resource
    private TaskRecordMapper taskRecordMapper;
    @Test
    public void recordTest(){
        QueryWrapper<TaskRecord> taskRecordQueryWrapper = new QueryWrapper<>();        
        taskRecordQueryWrapper.eq("order_no","XBC001005");
        TaskRecord taskRecord = taskRecordMapper.selectOne(taskRecordQueryWrapper);
        QueryWrapper<TaskRecord> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("task_status",0)
            .eq("work_center_no" ,taskRecord.getWorkCenterNo())                
            .ge("delivery_time",taskRecord.getDeliveryTime())                
            .ne("sub_order_no",taskRecord.getSubOrderNo())
            .orderByAsc("delivery_time")
            .last("limit 1");
        TaskRecord taskRecord1 = taskRecordMapper.selectOne(queryWrapper);        
        System.out.println(taskRecord1.getDeliveryTime().toLocalDateTime());
    }
}
```

