---
name: java-crud-generator
description: Java 开发专家技能，专为 Trae IDE AI 生成代码设计。基于数据表结构定义自动生成 Spring Boot 架构的 CRUD 代码，支持多种数据库 (Oracle/MySQL/SQLServer/PostgreSQL)、MyBatis/MyBatisPlus/Hibernate、参数校验、Swagger 文档、单元测试等。当用户需要生成 Java 后端 CRUD 代码时使用此技能。
---

# Java CRUD 代码生成专家

你是一名精通 Java Spring Boot 架构的开发专家，能够基于数据表结构定义自动生成高质量的 CRUD 代码。

## 核心能力

- **多数据库支持** - Oracle、MySQL (Percona/MariaDB)、SQL Server、PostgreSQL
- **ORM 框架** - 原生 MyBatis、MyBatis 通用 Mapper、MyBatis-Plus
- **代码分层** - Controller、Service、ServiceImpl、Mapper、Entity、DTO、VO
- **参数校验** - Hibernate Validator 注解
- **API 文档** - Swagger/OpenAPI 注解
- **服务调用** - Spring Cloud OpenFeign 消费端代码
- **测试生成** - JUnit 单元测试、CURL 测试脚本
- **导出支持** - Postman API JSON 定义

## 数据库类型映射

### 字段类型映射表

| 数据库类型 | MySQL | Oracle | SQL Server | PostgreSQL | Java 类型 |
|-----------|-------|--------|------------|------------|----------|
| 整数 | INT/BIGINT | NUMBER | INT/BIGINT | INTEGER/BIGINT | Integer/Long |
| 小数 | DECIMAL/NUMERIC | NUMBER | DECIMAL | NUMERIC | BigDecimal |
| 字符串 | VARCHAR/TEXT | VARCHAR2/CLOB | VARCHAR/NVARCHAR | VARCHAR/TEXT | String |
| 日期 | DATE/DATETIME | DATE | DATETIME | TIMESTAMP | Date/LocalDateTime |
| 布尔 | TINYINT(1) | NUMBER(1) | BIT | BOOLEAN | Boolean |
| 二进制 | BLOB | BLOB | VARBINARY | BYTEA | byte[] |

### 主键策略

```java
// MySQL - 自增主键
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// Oracle - 序列主键
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "seq_gen")
@SequenceGenerator(name = "seq_gen", sequenceName = "SEQ_TABLE_NAME", allocationSize = 1)
private Long id;

// PostgreSQL - 序列主键
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// SQL Server - 自增主键
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// UUID 主键
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private String id;
```

## 代码生成规则

### Entity 实体类

```java
package com.example.module.entity;

import com.baomidou.mybatisplus.annotation.*;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.Accessors;

import javax.validation.constraints.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * 用户信息表 Entity
 * 
 * @author generator
 * @since 2024-01-01
 */
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("sys_user")
@Schema(description = "用户信息表")
public class SysUser {

    /**
     * 主键 ID
     */
    @Schema(description = "主键 ID", example = "1")
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    /**
     * 用户名
     */
    @Schema(description = "用户名", example = "admin", required = true)
    @NotBlank(message = "用户名不能为空")
    @Size(max = 50, message = "用户名长度不能超过 50")
    @TableField("username")
    private String username;

    /**
     * 邮箱
     */
    @Schema(description = "邮箱", example = "admin@example.com")
    @Email(message = "邮箱格式不正确")
    @Size(max = 100, message = "邮箱长度不能超过 100")
    @TableField("email")
    private String email;

    /**
     * 年龄
     */
    @Schema(description = "年龄", example = "25")
    @Min(value = 0, message = "年龄不能小于 0")
    @Max(value = 150, message = "年龄不能大于 150")
    @TableField("age")
    private Integer age;

    /**
     * 余额
     */
    @Schema(description = "余额", example = "100.00")
    @DecimalMin(value = "0.00", message = "余额不能小于 0")
    @Digits(integer = 10, fraction = 2, message = "余额格式不正确")
    @TableField("balance")
    private BigDecimal balance;

    /**
     * 状态 0-禁用 1-启用
     */
    @Schema(description = "状态 0-禁用 1-启用", example = "1")
    @TableField("status")
    private Integer status;

    /**
     * 创建时间
     */
    @Schema(description = "创建时间")
    @TableField(value = "create_time", fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    /**
     * 更新时间
     */
    @Schema(description = "更新时间")
    @TableField(value = "update_time", fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    /**
     * 逻辑删除
     */
    @Schema(description = "逻辑删除")
    @TableLogic
    @TableField("deleted")
    private Integer deleted;
}
```

### Mapper 接口

```java
package com.example.module.mapper;

import com.example.module.entity.SysUser;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

import java.util.List;

/**
 * 用户信息表 Mapper 接口
 * 
 * @author generator
 * @since 2024-01-01
 */
@Mapper
public interface SysUserMapper extends BaseMapper<SysUser> {

    /**
     * 根据用户名查询用户
     * 
     * @param username 用户名
     * @return 用户信息
     */
    @Select("SELECT * FROM sys_user WHERE username = #{username} AND deleted = 0")
    SysUser selectByUsername(@Param("username") String username);

    /**
     * 查询所有启用的用户
     * 
     * @return 用户列表
     */
    @Select("SELECT * FROM sys_user WHERE status = 1 AND deleted = 0")
    List<SysUser> selectAllEnabled();
}
```

### XML Mapper (原生 MyBatis)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.module.mapper.SysUserMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.example.module.entity.SysUser">
        <id column="id" property="id" />
        <result column="username" property="username" />
        <result column="email" property="email" />
        <result column="age" property="age" />
        <result column="balance" property="balance" />
        <result column="status" property="status" />
        <result column="create_time" property="createTime" />
        <result column="update_time" property="updateTime" />
        <result column="deleted" property="deleted" />
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, username, email, age, balance, status, create_time, update_time, deleted
    </sql>

    <!-- 分页查询 -->
    <select id="selectPage" resultType="com.example.module.entity.SysUser">
        SELECT <include refid="Base_Column_List" />
        FROM sys_user
        WHERE deleted = 0
        <if test="username != null and username != ''">
            AND username LIKE CONCAT('%', #{username}, '%')
        </if>
        <if test="status != null">
            AND status = #{status}
        </if>
        ORDER BY create_time DESC
    </select>

    <!-- 根据 ID 查询 -->
    <select id="selectById" resultMap="BaseResultMap">
        SELECT <include refid="Base_Column_List" />
        FROM sys_user
        WHERE id = #{id} AND deleted = 0
    </select>

    <!-- 插入 -->
    <insert id="insert" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO sys_user (
            username, email, age, balance, status, create_time
        ) VALUES (
            #{username}, #{email}, #{age}, #{balance}, #{status}, #{createTime}
        )
    </insert>

    <!-- 更新 -->
    <update id="updateById">
        UPDATE sys_user
        SET username = #{username},
            email = #{email},
            age = #{age},
            balance = #{balance},
            status = #{status},
            update_time = #{updateTime}
        WHERE id = #{id}
    </update>

    <!-- 逻辑删除 -->
    <update id="deleteById">
        UPDATE sys_user
        SET deleted = 1, update_time = NOW()
        WHERE id = #{id}
    </update>
</mapper>
```

### Service 接口

```java
package com.example.module.service;

import com.example.module.entity.SysUser;
import com.baomidou.mybatisplus.extension.service.IService;
import com.example.module.dto.SysUserPageDTO;
import com.example.module.dto.SysUserCreateDTO;
import com.example.module.dto.SysUserUpdateDTO;
import com.example.module.vo.SysUserVO;

import java.util.List;

/**
 * 用户信息表 Service 接口
 * 
 * @author generator
 * @since 2024-01-01
 */
public interface SysUserService extends IService<SysUser> {

    /**
     * 分页查询用户列表
     * 
     * @param dto 查询参数
     * @return 分页结果
     */
    com.baomidou.mybatisplus.extension.plugins.pagination.Page<SysUserVO> page(SysUserPageDTO dto);

    /**
     * 根据 ID 查询用户详情
     * 
     * @param id 用户 ID
     * @return 用户详情
     */
    SysUserVO getById(Long id);

    /**
     * 创建用户
     * 
     * @param dto 创建参数
     * @return 用户 ID
     */
    Long create(SysUserCreateDTO dto);

    /**
     * 更新用户
     * 
     * @param dto 更新参数
     */
    void update(SysUserUpdateDTO dto);

    /**
     * 删除用户
     * 
     * @param id 用户 ID
     */
    void deleteById(Long id);

    /**
     * 根据用户名查询用户
     * 
     * @param username 用户名
     * @return 用户信息
     */
    SysUser getByUsername(String username);
}
```

### ServiceImpl 实现类

```java
package com.example.module.service.impl;

import com.example.module.entity.SysUser;
import com.example.module.mapper.SysUserMapper;
import com.example.module.service.SysUserService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.module.dto.SysUserPageDTO;
import com.example.module.dto.SysUserCreateDTO;
import com.example.module.dto.SysUserUpdateDTO;
import com.example.module.vo.SysUserVO;
import com.example.module.exception.BusinessException;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;

/**
 * 用户信息表 Service 实现类
 * 
 * @author generator
 * @since 2024-01-01
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class SysUserServiceImpl extends ServiceImpl<SysUserMapper, SysUser> implements SysUserService {

    private final SysUserMapper userMapper;

    @Override
    public com.baomidou.mybatisplus.extension.plugins.pagination.Page<SysUserVO> page(SysUserPageDTO dto) {
        var page = new com.baomidou.mybatisplus.extension.plugins.pagination.Page<SysUser>(
            dto.getPageNum(), dto.getPageSize()
        );
        
        var wrapper = new com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper<SysUser>()
            .eq(SysUser::getDeleted, 0)
            .like(dto.getUsername() != null, SysUser::getUsername, dto.getUsername())
            .eq(dto.getStatus() != null, SysUser::getStatus, dto.getStatus())
            .orderByDesc(SysUser::getCreateTime);
        
        var result = userMapper.selectPage(page, wrapper);
        
        // 转换为 VO
        var voPage = new com.baomidou.mybatisplus.extension.plugins.pagination.Page<SysUserVO>(
            result.getCurrent(), result.getSize(), result.getTotal()
        );
        var voList = result.getRecords().stream()
            .map(this::convertToVO)
            .toList();
        voPage.setRecords(voList);
        
        return voPage;
    }

    @Override
    public SysUserVO getById(Long id) {
        var entity = userMapper.selectById(id);
        if (entity == null || entity.getDeleted() == 1) {
            throw new BusinessException("用户不存在");
        }
        return convertToVO(entity);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long create(SysUserCreateDTO dto) {
        // 检查用户名是否已存在
        var existing = userMapper.selectByUsername(dto.getUsername());
        if (existing != null) {
            throw new BusinessException("用户名已存在");
        }
        
        var entity = new SysUser();
        BeanUtils.copyProperties(dto, entity);
        entity.setStatus(1);
        entity.setCreateTime(LocalDateTime.now());
        entity.setDeleted(0);
        
        userMapper.insert(entity);
        return entity.getId();
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void update(SysUserUpdateDTO dto) {
        var existing = userMapper.selectById(dto.getId());
        if (existing == null || existing.getDeleted() == 1) {
            throw new BusinessException("用户不存在");
        }
        
        // 检查用户名是否已被其他用户使用
        if (dto.getUsername() != null) {
            var userByName = userMapper.selectByUsername(dto.getUsername());
            if (userByName != null && !userByName.getId().equals(dto.getId())) {
                throw new BusinessException("用户名已存在");
            }
        }
        
        var entity = new SysUser();
        BeanUtils.copyProperties(dto, entity);
        entity.setUpdateTime(LocalDateTime.now());
        
        userMapper.updateById(entity);
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void deleteById(Long id) {
        var existing = userMapper.selectById(id);
        if (existing == null || existing.getDeleted() == 1) {
            throw new BusinessException("用户不存在");
        }
        
        existing.setDeleted(1);
        existing.setUpdateTime(LocalDateTime.now());
        userMapper.updateById(existing);
    }

    @Override
    public SysUser getByUsername(String username) {
        return userMapper.selectByUsername(username);
    }

    private SysUserVO convertToVO(SysUser entity) {
        var vo = new SysUserVO();
        BeanUtils.copyProperties(entity, vo);
        return vo;
    }
}
```

### Controller 控制器

```java
package com.example.module.controller;

import com.example.module.service.SysUserService;
import com.example.module.dto.SysUserPageDTO;
import com.example.module.dto.SysUserCreateDTO;
import com.example.module.dto.SysUserUpdateDTO;
import com.example.module.vo.SysUserVO;
import com.example.module.common.Result;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

/**
 * 用户信息表 Controller
 * 
 * @author generator
 * @since 2024-01-01
 */
@Tag(name = "用户管理", description = "用户信息表 CRUD 接口")
@RestController
@RequestMapping("/api/sys/user")
@RequiredArgsConstructor
public class SysUserController {

    private final SysUserService userService;

    /**
     * 分页查询用户列表
     */
    @Operation(summary = "分页查询用户列表")
    @GetMapping("/page")
    public Result<com.baomidou.mybatisplus.extension.plugins.pagination.Page<SysUserVO>> page(
            @Parameter(description = "查询参数") @Validated SysUserPageDTO dto) {
        var result = userService.page(dto);
        return Result.success(result);
    }

    /**
     * 根据 ID 查询用户详情
     */
    @Operation(summary = "根据 ID 查询用户详情")
    @GetMapping("/{id}")
    public Result<SysUserVO> getById(
            @Parameter(description = "用户 ID", example = "1") @PathVariable Long id) {
        var result = userService.getById(id);
        return Result.success(result);
    }

    /**
     * 创建用户
     */
    @Operation(summary = "创建用户")
    @PostMapping
    public Result<Long> create(
            @Parameter(description = "创建参数") @RequestBody @Validated SysUserCreateDTO dto) {
        var result = userService.create(dto);
        return Result.success(result);
    }

    /**
     * 更新用户
     */
    @Operation(summary = "更新用户")
    @PutMapping
    public Result<Void> update(
            @Parameter(description = "更新参数") @RequestBody @Validated SysUserUpdateDTO dto) {
        userService.update(dto);
        return Result.success();
    }

    /**
     * 删除用户
     */
    @Operation(summary = "删除用户")
    @DeleteMapping("/{id}")
    public Result<Void> deleteById(
            @Parameter(description = "用户 ID", example = "1") @PathVariable Long id) {
        userService.deleteById(id);
        return Result.success();
    }
}
```

### DTO 数据传输对象

```java
package com.example.module.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import javax.validation.constraints.*;
import java.math.BigDecimal;

/**
 * 用户信息表 DTO
 */
@Data
@Schema(description = "用户信息表 DTO")
public class SysUserDTO {

    @Schema(description = "用户 ID", example = "1")
    private Long id;

    @Schema(description = "用户名", example = "admin")
    @NotBlank(message = "用户名不能为空")
    @Size(max = 50, message = "用户名长度不能超过 50")
    private String username;

    @Schema(description = "邮箱", example = "admin@example.com")
    @Email(message = "邮箱格式不正确")
    @Size(max = 100, message = "邮箱长度不能超过 100")
    private String email;

    @Schema(description = "年龄", example = "25")
    @Min(value = 0, message = "年龄不能小于 0")
    @Max(value = 150, message = "年龄不能大于 150")
    private Integer age;

    @Schema(description = "余额", example = "100.00")
    @DecimalMin(value = "0.00", message = "余额不能小于 0")
    @Digits(integer = 10, fraction = 2, message = "余额格式不正确")
    private BigDecimal balance;

    @Schema(description = "状态 0-禁用 1-启用", example = "1")
    private Integer status;
}

/**
 * 分页查询 DTO
 */
@Data
@Schema(description = "用户分页查询参数")
class SysUserPageDTO extends SysUserDTO {

    @Schema(description = "页码", example = "1")
    @Min(value = 1, message = "页码不能小于 1")
    private Integer pageNum = 1;

    @Schema(description = "每页大小", example = "10")
    @Min(value = 1, message = "每页大小不能小于 1")
    @Max(value = 100, message = "每页大小不能大于 100")
    private Integer pageSize = 10;
}

/**
 * 创建 DTO
 */
@Data
@Schema(description = "用户创建参数")
class SysUserCreateDTO {

    @Schema(description = "用户名", example = "admin", required = true)
    @NotBlank(message = "用户名不能为空")
    @Size(max = 50, message = "用户名长度不能超过 50")
    private String username;

    @Schema(description = "邮箱", example = "admin@example.com")
    @Email(message = "邮箱格式不正确")
    private String email;

    @Schema(description = "年龄", example = "25")
    private Integer age;

    @Schema(description = "余额", example = "100.00")
    private BigDecimal balance;
}

/**
 * 更新 DTO
 */
@Data
@Schema(description = "用户更新参数")
class SysUserUpdateDTO {

    @Schema(description = "用户 ID", example = "1", required = true)
    @NotNull(message = "用户 ID 不能为空")
    private Long id;

    @Schema(description = "用户名", example = "admin")
    @Size(max = 50, message = "用户名长度不能超过 50")
    private String username;

    @Schema(description = "邮箱", example = "admin@example.com")
    @Email(message = "邮箱格式不正确")
    private String email;

    @Schema(description = "年龄", example = "25")
    private Integer age;

    @Schema(description = "余额", example = "100.00")
    private BigDecimal balance;

    @Schema(description = "状态 0-禁用 1-启用", example = "1")
    private Integer status;
}
```

### VO 视图对象

```java
package com.example.module.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * 用户信息表 VO
 */
@Data
@Schema(description = "用户信息 VO")
public class SysUserVO {

    @Schema(description = "用户 ID", example = "1")
    private Long id;

    @Schema(description = "用户名", example = "admin")
    private String username;

    @Schema(description = "邮箱", example = "admin@example.com")
    private String email;

    @Schema(description = "年龄", example = "25")
    private Integer age;

    @Schema(description = "余额", example = "100.00")
    private BigDecimal balance;

    @Schema(description = "状态 0-禁用 1-启用", example = "1")
    private Integer status;

    @Schema(description = "创建时间")
    private LocalDateTime createTime;

    @Schema(description = "更新时间")
    private LocalDateTime updateTime;
}
```

## OpenFeign 服务消费端

```java
package com.example.module.client;

import com.example.module.vo.SysUserVO;
import com.example.module.common.Result;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

/**
 * 用户服务 Feign 客户端
 */
@FeignClient(name = "user-service", path = "/api/sys/user")
public interface SysUserClient {

    /**
     * 根据 ID 查询用户
     */
    @GetMapping("/{id}")
    Result<SysUserVO> getById(@PathVariable("id") Long id);

    /**
     * 创建用户
     */
    @PostMapping
    Result<Long> create(@RequestBody SysUserCreateDTO dto);

    /**
     * 更新用户
     */
    @PutMapping
    Result<Void> update(@RequestBody SysUserUpdateDTO dto);

    /**
     * 删除用户
     */
    @DeleteMapping("/{id}")
    Result<Void> deleteById(@PathVariable("id") Long id);
}
```

## JUnit 单元测试

```java
package com.example.module.service;

import com.example.module.dto.SysUserCreateDTO;
import com.example.module.dto.SysUserUpdateDTO;
import com.example.module.vo.SysUserVO;
import com.example.module.common.Result;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;

import static org.junit.jupiter.api.Assertions.*;

/**
 * 用户服务单元测试
 */
@SpringBootTest
@Transactional
class SysUserServiceTest {

    @Autowired
    private SysUserService userService;

    @Test
    @DisplayName("创建用户 - 成功")
    void testCreate_Success() {
        // 准备数据
        var dto = new SysUserCreateDTO();
        dto.setUsername("test_user");
        dto.setEmail("test@example.com");
        dto.setAge(25);
        dto.setBalance(new BigDecimal("100.00"));

        // 执行
        var userId = userService.create(dto);

        // 验证
        assertNotNull(userId);
        var user = userService.getById(userId);
        assertEquals("test_user", user.getUsername());
        assertEquals("test@example.com", user.getEmail());
    }

    @Test
    @DisplayName("创建用户 - 用户名已存在")
    void testCreate_UsernameExists() {
        // 准备数据
        var dto = new SysUserCreateDTO();
        dto.setUsername("existing_user");
        dto.setEmail("existing@example.com");

        // 先创建一个
        userService.create(dto);

        // 再次创建相同用户名
        var dto2 = new SysUserCreateDTO();
        dto2.setUsername("existing_user");
        dto2.setEmail("another@example.com");

        // 验证抛出异常
        assertThrows(RuntimeException.class, () -> userService.create(dto2));
    }

    @Test
    @DisplayName("更新用户 - 成功")
    void testUpdate_Success() {
        // 准备数据
        var createDto = new SysUserCreateDTO();
        createDto.setUsername("update_test");
        createDto.setEmail("update@example.com");
        var userId = userService.create(createDto);

        // 更新
        var updateDto = new SysUserUpdateDTO();
        updateDto.setId(userId);
        updateDto.setUsername("updated_user");
        updateDto.setEmail("updated@example.com");
        userService.update(updateDto);

        // 验证
        var user = userService.getById(userId);
        assertEquals("updated_user", user.getUsername());
        assertEquals("updated@example.com", user.getEmail());
    }

    @Test
    @DisplayName("删除用户 - 成功")
    void testDelete_Success() {
        // 准备数据
        var dto = new SysUserCreateDTO();
        dto.setUsername("delete_test");
        var userId = userService.create(dto);

        // 删除
        userService.deleteById(userId);

        // 验证 - 查询应抛出异常或返回 null
        assertThrows(RuntimeException.class, () -> userService.getById(userId));
    }

    @Test
    @DisplayName("分页查询 - 成功")
    void testPage_Success() {
        // 准备数据
        for (int i = 0; i < 15; i++) {
            var dto = new SysUserCreateDTO();
            dto.setUsername("page_test_" + i);
            dto.setEmail("page_" + i + "@example.com");
            userService.create(dto);
        }

        // 分页查询
        var pageDto = new SysUserPageDTO();
        pageDto.setPageNum(1);
        pageDto.setPageSize(10);
        var result = userService.page(pageDto);

        // 验证
        assertEquals(1, result.getCurrent());
        assertEquals(10, result.getSize());
        assertEquals(15, result.getTotal());
        assertEquals(10, result.getRecords().size());
    }
}
```

## CURL 测试脚本

```bash
#!/bin/bash
# 用户管理 API 测试脚本

BASE_URL="http://localhost:8080/api/sys/user"

echo "=== 创建用户 ==="
curl -X POST "$BASE_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test_user",
    "email": "test@example.com",
    "age": 25,
    "balance": 100.00
  }'

echo -e "\n\n=== 分页查询 ==="
curl -X GET "$BASE_URL/page?pageNum=1&pageSize=10"

echo -e "\n\n=== 根据 ID 查询 ==="
curl -X GET "$BASE_URL/1"

echo -e "\n\n=== 更新用户 ==="
curl -X PUT "$BASE_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "username": "updated_user",
    "email": "updated@example.com",
    "age": 26,
    "balance": 200.00,
    "status": 1
  }'

echo -e "\n\n=== 删除用户 ==="
curl -X DELETE "$BASE_URL/1"
```

## Postman API 定义

```json
{
  "info": {
    "name": "用户管理 API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "用户管理",
      "item": [
        {
          "name": "分页查询用户列表",
          "request": {
            "method": "GET",
            "url": {
              "raw": "{{baseUrl}}/api/sys/user/page?pageNum=1&pageSize=10",
              "host": ["{{baseUrl}}"],
              "path": ["api", "sys", "user", "page"],
              "query": [
                {"key": "pageNum", "value": "1"},
                {"key": "pageSize", "value": "10"}
              ]
            }
          }
        },
        {
          "name": "根据 ID 查询用户",
          "request": {
            "method": "GET",
            "url": {
              "raw": "{{baseUrl}}/api/sys/user/:id",
              "host": ["{{baseUrl}}"],
              "path": ["api", "sys", "user", ":id"],
              "variable": [{"key": "id", "value": "1"}]
            }
          }
        },
        {
          "name": "创建用户",
          "request": {
            "method": "POST",
            "header": [{"key": "Content-Type", "value": "application/json"}],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"username\": \"admin\",\n  \"email\": \"admin@example.com\",\n  \"age\": 25,\n  \"balance\": 100.00\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/sys/user",
              "host": ["{{baseUrl}}"],
              "path": ["api", "sys", "user"]
            }
          }
        },
        {
          "name": "更新用户",
          "request": {
            "method": "PUT",
            "header": [{"key": "Content-Type", "value": "application/json"}],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"id\": 1,\n  \"username\": \"updated\",\n  \"email\": \"updated@example.com\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/sys/user",
              "host": ["{{baseUrl}}"],
              "path": ["api", "sys", "user"]
            }
          }
        },
        {
          "name": "删除用户",
          "request": {
            "method": "DELETE",
            "url": {
              "raw": "{{baseUrl}}/api/sys/user/:id",
              "host": ["{{baseUrl}}"],
              "path": ["api", "sys", "user", ":id"],
              "variable": [{"key": "id", "value": "1"}]
            }
          }
        }
      ]
    }
  ],
  "variable": [{"key": "baseUrl", "value": "http://localhost:8080"}]
}
```

## 数据库表结构解析

### MySQL 表结构示例

```sql
CREATE TABLE `sys_user` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键 ID',
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `email` VARCHAR(100) DEFAULT NULL COMMENT '邮箱',
  `age` INT DEFAULT NULL COMMENT '年龄',
  `balance` DECIMAL(10,2) DEFAULT NULL COMMENT '余额',
  `status` TINYINT DEFAULT 1 COMMENT '状态 0-禁用 1-启用',
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `deleted` TINYINT DEFAULT 0 COMMENT '逻辑删除 0-未删除 1-已删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`),
  KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';
```

### Oracle 表结构示例

```sql
CREATE TABLE sys_user (
  id NUMBER(19) NOT NULL,
  username VARCHAR2(50) NOT NULL,
  email VARCHAR2(100),
  age NUMBER(10),
  balance NUMBER(10,2),
  status NUMBER(1) DEFAULT 1,
  create_time TIMESTAMP DEFAULT SYSTIMESTAMP,
  update_time TIMESTAMP DEFAULT SYSTIMESTAMP,
  deleted NUMBER(1) DEFAULT 0,
  CONSTRAINT pk_sys_user PRIMARY KEY (id),
  CONSTRAINT uk_username UNIQUE (username)
);

COMMENT ON TABLE sys_user IS '用户信息表';
COMMENT ON COLUMN sys_user.id IS '主键 ID';
COMMENT ON COLUMN sys_user.username IS '用户名';
COMMENT ON COLUMN sys_user.email IS '邮箱';

-- 创建序列
CREATE SEQUENCE seq_sys_user START WITH 1 INCREMENT BY 1;

-- 创建触发器
CREATE OR REPLACE TRIGGER trg_sys_user
BEFORE INSERT ON sys_user
FOR EACH ROW
BEGIN
  SELECT seq_sys_user.NEXTVAL INTO :NEW.id FROM DUAL;
END;
```

## 代码生成流程

1. **解析表结构** - 读取数据库表定义，获取字段名、类型、长度、注释
2. **识别主键** - 检测主键字段，确定生成策略 (自增/序列/UUID)
3. **识别索引** - 检测唯一索引，添加相应校验
4. **类型映射** - 根据数据库类型映射到 Java 类型
5. **生成 Entity** - 生成 JPA/MyBatis 实体类
6. **生成 Mapper** - 生成数据访问层接口和 XML
7. **生成 Service** - 生成业务逻辑层接口和实现
8. **生成 Controller** - 生成 RESTful API 控制器
9. **生成 DTO/VO** - 生成数据传输对象和视图对象
10. **生成测试** - 生成 JUnit 测试和 CURL 脚本

## 触发此技能

当用户需要以下帮助时主动使用此技能：

- 基于数据库表生成 CRUD 代码
- 创建 Spring Boot 项目结构
- 生成 MyBatis/MyBatis-Plus 代码
- 生成参数校验注解
- 生成 Swagger API 文档注解
- 生成 OpenFeign 服务调用代码
- 生成 JUnit 单元测试
- 生成 Postman API 集合
- 生成 CURL 测试脚本
- 多数据库类型代码适配
