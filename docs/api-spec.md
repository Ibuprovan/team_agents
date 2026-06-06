# 学生成绩管理系统 - API 接口规范

> **文档版本：** V1.0  
> **创建日期：** 2026-06-05  
> **架构师：** Architect Agent  
> **文档状态：** 已评审

---

## 1. API 概述

### 1.1 基础信息

| 项目 | 说明 |
|------|------|
| **基础路径** | `/api/v1` |
| **协议** | HTTP/HTTPS |
| **数据格式** | JSON |
| **字符编码** | UTF-8 |
| **认证方式** | 暂无（MVP 阶段） |

### 1.2 通用响应格式

#### 成功响应

```json
{
    "success": true,
    "data": {
        // 业务数据
    },
    "message": "操作成功"
}
```

#### 列表响应（分页）

```json
{
    "success": true,
    "data": {
        "items": [],
        "total": 100,
        "page": 1,
        "page_size": 20,
        "total_pages": 5
    }
}
```

#### 错误响应

```json
{
    "success": false,
    "error": {
        "code": "NOT_FOUND",
        "message": "学生 '20260001' 不存在"
    }
}
```

### 1.3 HTTP 状态码

| 状态码 | 说明 | 使用场景 |
|--------|------|---------|
| 200 | OK | 查询成功、更新成功 |
| 201 | Created | 创建成功 |
| 204 | No Content | 删除成功 |
| 400 | Bad Request | 请求参数错误 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 数据冲突（如学号重复） |
| 422 | Unprocessable Entity | 数据验证失败 |
| 500 | Internal Server Error | 服务器内部错误 |

### 1.4 错误码定义

| 错误码 | 说明 | HTTP 状态码 |
|--------|------|-------------|
| `NOT_FOUND` | 资源不存在 | 404 |
| `DUPLICATE` | 数据重复 | 409 |
| `VALIDATION_ERROR` | 数据验证失败 | 422 |
| `INVALID_FORMAT` | 格式错误 | 400 |
| `INTERNAL_ERROR` | 内部错误 | 500 |

---

## 2. 学生信息接口

### 2.1 添加学生

**POST** `/api/v1/students`

创建新的学生记录。

#### 请求参数

```json
{
    "student_id": "20260001",
    "name": "张三",
    "gender": "男",
    "class_name": "三年一班",
    "enrollment_year": 2026
}
```

| 字段 | 类型 | 必填 | 说明 | 校验规则 |
|------|------|------|------|---------|
| student_id | string | 是 | 学号 | 8位数字，格式：YYYY + 4位序号 |
| name | string | 是 | 姓名 | 2-20个字符 |
| gender | string | 是 | 性别 | "男" 或 "女" |
| class_name | string | 是 | 班级 | 2-20个字符 |
| enrollment_year | integer | 是 | 入学年份 | 2000-2100 |

#### 成功响应

```json
{
    "success": true,
    "data": {
        "student_id": "20260001",
        "name": "张三",
        "gender": "男",
        "class_name": "三年一班",
        "enrollment_year": 2026,
        "created_at": "2026-06-05T10:00:00",
        "updated_at": "2026-06-05T10:00:00"
    },
    "message": "学生添加成功"
}
```

#### 错误响应

```json
// 学号重复
{
    "success": false,
    "error": {
        "code": "DUPLICATE",
        "message": "学生 的 学号 '20260001' 已存在"
    }
}

// 数据验证失败
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "学号格式错误，应为8位数字（如：20260001）"
    }
}
```

---

### 2.2 查询学生列表

**GET** `/api/v1/students`

获取学生列表，支持分页和筛选。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| page | integer | 否 | 页码（默认：1） | `?page=1` |
| page_size | integer | 否 | 每页数量（默认：20，最大：100） | `?page_size=10` |
| class_name | string | 否 | 按班级筛选 | `?class_name=三年一班` |
| keyword | string | 否 | 按学号或姓名搜索 | `?keyword=张三` |

#### 请求示例

```
GET /api/v1/students?page=1&page_size=10&class_name=三年一班
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "items": [
            {
                "student_id": "20260001",
                "name": "张三",
                "gender": "男",
                "class_name": "三年一班",
                "enrollment_year": 2026,
                "created_at": "2026-06-05T10:00:00",
                "updated_at": "2026-06-05T10:00:00"
            }
        ],
        "total": 30,
        "page": 1,
        "page_size": 10,
        "total_pages": 3
    }
}
```

---

### 2.3 查询单个学生

**GET** `/api/v1/students/{student_id}`

根据学号查询学生详细信息。

#### 路径参数

| 参数 | 类型 | 说明 | 示例 |
|------|------|------|------|
| student_id | string | 学号 | `20260001` |

#### 请求示例

```
GET /api/v1/students/20260001
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "student_id": "20260001",
        "name": "张三",
        "gender": "男",
        "class_name": "三年一班",
        "enrollment_year": 2026,
        "created_at": "2026-06-05T10:00:00",
        "updated_at": "2026-06-05T10:00:00"
    }
}
```

#### 错误响应

```json
{
    "success": false,
    "error": {
        "code": "NOT_FOUND",
        "message": "学生 '20260001' 不存在"
    }
}
```

---

### 2.4 更新学生信息

**PUT** `/api/v1/students/{student_id}`

更新学生基本信息。

#### 路径参数

| 参数 | 类型 | 说明 |
|------|------|------|
| student_id | string | 学号 |

#### 请求参数

```json
{
    "name": "张三丰",
    "class_name": "三年二班"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 否 | 姓名（2-20个字符） |
| gender | string | 否 | 性别（"男" 或 "女"） |
| class_name | string | 否 | 班级 |
| enrollment_year | integer | 否 | 入学年份 |

#### 成功响应

```json
{
    "success": true,
    "data": {
        "student_id": "20260001",
        "name": "张三丰",
        "gender": "男",
        "class_name": "三年二班",
        "enrollment_year": 2026,
        "created_at": "2026-06-05T10:00:00",
        "updated_at": "2026-06-05T11:00:00"
    },
    "message": "学生信息更新成功"
}
```

---

### 2.5 删除学生

**DELETE** `/api/v1/students/{student_id}`

删除学生记录及其所有成绩。

#### 路径参数

| 参数 | 类型 | 说明 |
|------|------|------|
| student_id | string | 学号 |

#### 请求示例

```
DELETE /api/v1/students/20260001
```

#### 成功响应

```json
{
    "success": true,
    "message": "学生 '20260001' 删除成功"
}
```

---

## 3. 成绩管理接口

### 3.1 录入单条成绩

**POST** `/api/v1/grades`

录入单个学生的单科成绩。

#### 请求参数

```json
{
    "student_id": "20260001",
    "subject": "数学",
    "score": 95.5,
    "exam_type": "期中",
    "exam_date": "2026-04-15"
}
```

| 字段 | 类型 | 必填 | 说明 | 校验规则 |
|------|------|------|------|---------|
| student_id | string | 是 | 学号 | 必须存在 |
| subject | string | 是 | 科目 | 语文/数学/英语/物理/化学/生物/历史/地理/政治 |
| score | number | 是 | 分数 | 0-100，支持1位小数 |
| exam_type | string | 是 | 考试类型 | 期中/期末/月考/单元测试 |
| exam_date | string | 是 | 考试日期 | 格式：YYYY-MM-DD |

#### 成功响应

```json
{
    "success": true,
    "data": {
        "grade_id": 1,
        "student_id": "20260001",
        "subject": "数学",
        "score": 95.5,
        "exam_type": "期中",
        "exam_date": "2026-04-15",
        "created_at": "2026-06-05T10:00:00"
    },
    "message": "成绩录入成功"
}
```

#### 错误响应

```json
// 学生不存在
{
    "success": false,
    "error": {
        "code": "NOT_FOUND",
        "message": "学生 '20260099' 不存在"
    }
}

// 成绩重复
{
    "success": false,
    "error": {
        "code": "DUPLICATE",
        "message": "成绩 的 学生-科目-考试类型 '20260001-数学-期中' 已存在"
    }
}

// 分数超范围
{
    "success": false,
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "分数 150 超出范围（0-100）"
    }
}
```

---

### 3.2 批量录入成绩

**POST** `/api/v1/grades/batch`

批量录入多个学生的成绩。

#### 请求参数

```json
{
    "subject": "数学",
    "exam_type": "期中",
    "exam_date": "2026-04-15",
    "grades": [
        {"student_id": "20260001", "score": 95.5},
        {"student_id": "20260002", "score": 88.0},
        {"student_id": "20260003", "score": 72.5}
    ]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| subject | string | 是 | 科目（所有成绩同一科目） |
| exam_type | string | 是 | 考试类型 |
| exam_date | string | 是 | 考试日期 |
| grades | array | 是 | 成绩列表 |
| grades[].student_id | string | 是 | 学号 |
| grades[].score | number | 是 | 分数 |

#### 成功响应

```json
{
    "success": true,
    "data": {
        "total": 3,
        "success_count": 3,
        "fail_count": 0,
        "results": [
            {"student_id": "20260001", "status": "success", "grade_id": 1},
            {"student_id": "20260002", "status": "success", "grade_id": 2},
            {"student_id": "20260003", "status": "success", "grade_id": 3}
        ]
    },
    "message": "批量录入完成：成功 3 条，失败 0 条"
}
```

#### 部分失败响应

```json
{
    "success": true,
    "data": {
        "total": 3,
        "success_count": 2,
        "fail_count": 1,
        "results": [
            {"student_id": "20260001", "status": "success", "grade_id": 1},
            {"student_id": "20260002", "status": "success", "grade_id": 2},
            {"student_id": "20260099", "status": "fail", "error": "学生不存在"}
        ]
    },
    "message": "批量录入完成：成功 2 条，失败 1 条"
}
```

---

### 3.3 查询成绩列表

**GET** `/api/v1/grades`

查询成绩列表，支持多维度筛选。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| student_id | string | 否 | 按学号筛选 | `?student_id=20260001` |
| class_name | string | 否 | 按班级筛选 | `?class_name=三年一班` |
| subject | string | 否 | 按科目筛选 | `?subject=数学` |
| exam_type | string | 否 | 按考试类型筛选 | `?exam_type=期中` |
| page | integer | 否 | 页码 | `?page=1` |
| page_size | integer | 否 | 每页数量 | `?page_size=20` |

#### 请求示例

```
GET /api/v1/grades?class_name=三年一班&subject=数学&exam_type=期中
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "items": [
            {
                "grade_id": 1,
                "student_id": "20260001",
                "student_name": "张三",
                "class_name": "三年一班",
                "subject": "数学",
                "score": 95.5,
                "exam_type": "期中",
                "exam_date": "2026-04-15"
            },
            {
                "grade_id": 2,
                "student_id": "20260002",
                "student_name": "李四",
                "class_name": "三年一班",
                "subject": "数学",
                "score": 88.0,
                "exam_type": "期中",
                "exam_date": "2026-04-15"
            }
        ],
        "total": 30,
        "page": 1,
        "page_size": 20,
        "total_pages": 2
    }
}
```

---

### 3.4 更新成绩

**PUT** `/api/v1/grades/{grade_id}`

修改已录入的成绩。

#### 路径参数

| 参数 | 类型 | 说明 |
|------|------|------|
| grade_id | integer | 成绩ID |

#### 请求参数

```json
{
    "score": 92.0
}
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "grade_id": 1,
        "student_id": "20260001",
        "subject": "数学",
        "score": 92.0,
        "exam_type": "期中",
        "exam_date": "2026-04-15",
        "updated_at": "2026-06-05T11:00:00"
    },
    "message": "成绩更新成功"
}
```

---

### 3.5 删除成绩

**DELETE** `/api/v1/grades/{grade_id}`

删除成绩记录。

#### 路径参数

| 参数 | 类型 | 说明 |
|------|------|------|
| grade_id | integer | 成绩ID |

#### 成功响应

```json
{
    "success": true,
    "message": "成绩记录删除成功"
}
```

---

## 4. 统计分析接口

### 4.1 获取统计数据

**GET** `/api/v1/statistics`

获取成绩统计数据，支持多种统计指标。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| class_name | string | 否 | 按班级统计 | `?class_name=三年一班` |
| subject | string | 否 | 按科目统计 | `?subject=数学` |
| exam_type | string | 否 | 按考试类型统计 | `?exam_type=期中` |
| metrics | string | 否 | 统计指标（逗号分隔） | `?metrics=avg,max,min,pass_rate` |

#### 可选统计指标

| 指标 | 说明 |
|------|------|
| avg | 平均分 |
| max | 最高分 |
| min | 最低分 |
| pass_rate | 及格率（≥60分） |
| excellent_rate | 优秀率（≥90分） |
| median | 中位数 |
| std_dev | 标准差 |

#### 请求示例

```
GET /api/v1/statistics?class_name=三年一班&subject=数学&exam_type=期中&metrics=avg,max,min,pass_rate
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "query": {
            "class_name": "三年一班",
            "subject": "数学",
            "exam_type": "期中"
        },
        "total_students": 30,
        "metrics": {
            "avg": 78.5,
            "max": 98.0,
            "min": 45.0,
            "pass_rate": 83.33
        }
    }
}
```

---

### 4.2 获取排名数据

**GET** `/api/v1/statistics/ranking`

获取成绩排名列表。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| class_name | string | 条件必填 | 班级排名时必填 | `?class_name=三年一班` |
| subject | string | 是 | 科目 | `?subject=数学` |
| exam_type | string | 是 | 考试类型 | `?exam_type=期中` |
| scope | string | 否 | 排名范围（class/grade） | `?scope=class` |
| limit | integer | 否 | 返回数量（默认：全部） | `?limit=10` |

#### 请求示例

```
GET /api/v1/statistics/ranking?class_name=三年一班&subject=数学&exam_type=期中&scope=class&limit=10
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "query": {
            "class_name": "三年一班",
            "subject": "数学",
            "exam_type": "期中",
            "scope": "class"
        },
        "rankings": [
            {
                "rank": 1,
                "student_id": "20260015",
                "student_name": "王五",
                "score": 98.0
            },
            {
                "rank": 2,
                "student_id": "20260001",
                "student_name": "张三",
                "score": 95.5
            },
            {
                "rank": 3,
                "student_id": "20260008",
                "student_name": "赵六",
                "score": 92.0
            }
        ]
    }
}
```

---

### 4.3 获取学生综合统计

**GET** `/api/v1/statistics/student/{student_id}`

获取指定学生的综合统计数据。

#### 路径参数

| 参数 | 类型 | 说明 |
|------|------|------|
| student_id | string | 学号 |

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| exam_type | string | 否 | 按考试类型筛选 |

#### 请求示例

```
GET /api/v1/statistics/student/20260001?exam_type=期中
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "student_id": "20260001",
        "student_name": "张三",
        "class_name": "三年一班",
        "exam_type": "期中",
        "subjects": [
            {
                "subject": "语文",
                "score": 85.0,
                "class_rank": 5,
                "grade_rank": 20
            },
            {
                "subject": "数学",
                "score": 95.5,
                "class_rank": 2,
                "grade_rank": 10
            },
            {
                "subject": "英语",
                "score": 88.0,
                "class_rank": 8,
                "grade_rank": 35
            }
        ],
        "total_score": 268.5,
        "average_score": 89.5,
        "class_rank_total": 3,
        "grade_rank_total": 15
    }
}
```

---

## 5. 数据导入导出接口

### 5.1 导入成绩数据

**POST** `/api/v1/import/grades`

从 CSV 文件批量导入成绩。

#### 请求格式

`Content-Type: multipart/form-data`

#### 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | file | 是 | CSV 文件 |
| exam_type | string | 是 | 考试类型 |
| exam_date | string | 是 | 考试日期 |

#### CSV 文件格式

```csv
student_id,subject,score
20260001,语文,85.0
20260001,数学,95.5
20260002,语文,78.0
```

#### 成功响应

```json
{
    "success": true,
    "data": {
        "total_rows": 3,
        "success_count": 3,
        "fail_count": 0,
        "errors": []
    },
    "message": "导入完成：成功 3 条，失败 0 条"
}
```

---

### 5.2 导出成绩数据

**GET** `/api/v1/export/grades`

导出成绩数据为 CSV 文件。

#### 查询参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| class_name | string | 否 | 按班级导出 |
| subject | string | 否 | 按科目导出 |
| exam_type | string | 否 | 按考试类型导出 |
| format | string | 否 | 导出格式（csv/json），默认：csv |

#### 请求示例

```
GET /api/v1/export/grades?class_name=三年一班&exam_type=期中&format=csv
```

#### 响应

`Content-Type: text/csv`

```csv
学号,姓名,班级,科目,分数,考试类型,考试日期
20260001,张三,三年一班,语文,85.0,期中,2026-04-15
20260001,张三,三年一班,数学,95.5,期中,2026-04-15
20260002,李四,三年一班,语文,78.0,期中,2026-04-15
```

---

## 6. 健康检查接口

### 6.1 服务健康检查

**GET** `/api/v1/health`

检查服务是否正常运行。

#### 成功响应

```json
{
    "success": true,
    "data": {
        "status": "healthy",
        "version": "1.0.0",
        "database": "connected",
        "timestamp": "2026-06-05T10:00:00"
    }
}
```

---

## 7. Pydantic 模式定义

### 7.1 学生相关模式

```python
# src/schemas/student.py
from pydantic import BaseModel, Field, field_validator
from datetime import datetime
from typing import Optional
import re

class StudentBase(BaseModel):
    """学生基础模式"""
    student_id: str = Field(..., description="学号（8位数字）")
    name: str = Field(..., min_length=2, max_length=20, description="姓名")
    gender: str = Field(..., description="性别")
    class_name: str = Field(..., min_length=2, max_length=20, description="班级")
    enrollment_year: int = Field(..., ge=2000, le=2100, description="入学年份")

    @field_validator('student_id')
    @classmethod
    def validate_student_id(cls, v):
        if not re.match(r"^\d{4}\d{4}$", v):
            raise ValueError("学号格式错误，应为8位数字（如：20260001）")
        return v

    @field_validator('gender')
    @classmethod
    def validate_gender(cls, v):
        if v not in ["男", "女"]:
            raise ValueError("性别只能是 '男' 或 '女'")
        return v

class StudentCreate(StudentBase):
    """创建学生请求模式"""
    pass

class StudentUpdate(BaseModel):
    """更新学生请求模式"""
    name: Optional[str] = Field(None, min_length=2, max_length=20)
    gender: Optional[str] = None
    class_name: Optional[str] = Field(None, min_length=2, max_length=20)
    enrollment_year: Optional[int] = Field(None, ge=2000, le=2100)

    @field_validator('gender')
    @classmethod
    def validate_gender(cls, v):
        if v is not None and v not in ["男", "女"]:
            raise ValueError("性别只能是 '男' 或 '女'")
        return v

class StudentResponse(StudentBase):
    """学生响应模式"""
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True
```

### 7.2 成绩相关模式

```python
# src/schemas/grade.py
from pydantic import BaseModel, Field, field_validator
from datetime import date, datetime
from typing import Optional, List

class GradeBase(BaseModel):
    """成绩基础模式"""
    student_id: str = Field(..., description="学号")
    subject: str = Field(..., description="科目")
    score: float = Field(..., ge=0, le=100, description="分数（0-100）")
    exam_type: str = Field(..., description="考试类型")
    exam_date: date = Field(..., description="考试日期")

    @field_validator('subject')
    @classmethod
    def validate_subject(cls, v):
        valid_subjects = ["语文", "数学", "英语", "物理", "化学", "生物", "历史", "地理", "政治"]
        if v not in valid_subjects:
            raise ValueError(f"科目必须是以下之一：{', '.join(valid_subjects)}")
        return v

    @field_validator('exam_type')
    @classmethod
    def validate_exam_type(cls, v):
        valid_types = ["期中", "期末", "月考", "单元测试"]
        if v not in valid_types:
            raise ValueError(f"考试类型必须是以下之一：{', '.join(valid_types)}")
        return v

    @field_validator('score')
    @classmethod
    def validate_score(cls, v):
        # 支持1位小数
        if round(v, 1) != v:
            raise ValueError("分数最多支持1位小数")
        return v

class GradeCreate(GradeBase):
    """创建成绩请求模式"""
    pass

class GradeBatchItem(BaseModel):
    """批量录入单项"""
    student_id: str
    score: float = Field(..., ge=0, le=100)

class GradeBatchCreate(BaseModel):
    """批量创建成绩请求模式"""
    subject: str
    exam_type: str
    exam_date: date
    grades: List[GradeBatchItem]

class GradeUpdate(BaseModel):
    """更新成绩请求模式"""
    score: float = Field(..., ge=0, le=100)

class GradeResponse(GradeBase):
    """成绩响应模式"""
    grade_id: int
    student_name: Optional[str] = None
    class_name: Optional[str] = None
    created_at: datetime
    updated_at: Optional[datetime] = None

    class Config:
        from_attributes = True
```

### 7.3 统计相关模式

```python
# src/schemas/statistics.py
from pydantic import BaseModel, Field
from typing import Optional, List
from enum import Enum

class StatisticsMetric(str, Enum):
    """统计指标枚举"""
    AVG = "avg"
    MAX = "max"
    MIN = "min"
    PASS_RATE = "pass_rate"
    EXCELLENT_RATE = "excellent_rate"
    MEDIAN = "median"
    STD_DEV = "std_dev"

class StatisticsQuery(BaseModel):
    """统计查询模式"""
    class_name: Optional[str] = None
    subject: Optional[str] = None
    exam_type: Optional[str] = None
    metrics: List[StatisticsMetric] = Field(
        default=[StatisticsMetric.AVG, StatisticsMetric.MAX, StatisticsMetric.MIN, StatisticsMetric.PASS_RATE]
    )

class StatisticsResponse(BaseModel):
    """统计响应模式"""
    query: StatisticsQuery
    total_students: int
    metrics: dict

class RankingItem(BaseModel):
    """排名项"""
    rank: int
    student_id: str
    student_name: str
    score: float

class RankingResponse(BaseModel):
    """排名响应模式"""
    query: dict
    rankings: List[RankingItem]

class StudentSubjectStats(BaseModel):
    """学生单科统计"""
    subject: str
    score: float
    class_rank: Optional[int] = None
    grade_rank: Optional[int] = None

class StudentStatisticsResponse(BaseModel):
    """学生综合统计响应"""
    student_id: str
    student_name: str
    class_name: str
    exam_type: Optional[str] = None
    subjects: List[StudentSubjectStats]
    total_score: float
    average_score: float
    class_rank_total: Optional[int] = None
    grade_rank_total: Optional[int] = None
```

### 7.4 通用响应模式

```python
# src/schemas/common.py
from pydantic import BaseModel, Field
from typing import Optional, Any, List, Generic, TypeVar
from enum import Enum

T = TypeVar('T')

class ErrorCode(str, Enum):
    """错误码枚举"""
    NOT_FOUND = "NOT_FOUND"
    DUPLICATE = "DUPLICATE"
    VALIDATION_ERROR = "VALIDATION_ERROR"
    INVALID_FORMAT = "INVALID_FORMAT"
    INTERNAL_ERROR = "INTERNAL_ERROR"

class ErrorDetail(BaseModel):
    """错误详情"""
    code: ErrorCode
    message: str

class ApiResponse(BaseModel):
    """通用 API 响应"""
    success: bool
    data: Optional[Any] = None
    message: Optional[str] = None
    error: Optional[ErrorDetail] = None

class PaginatedData(BaseModel):
    """分页数据"""
    items: List[Any]
    total: int
    page: int
    page_size: int
    total_pages: int

class PaginatedResponse(BaseModel):
    """分页响应"""
    success: bool = True
    data: PaginatedData

class BatchResultItem(BaseModel):
    """批量操作结果项"""
    student_id: str
    status: str  # "success" 或 "fail"
    grade_id: Optional[int] = None
    error: Optional[str] = None

class BatchResult(BaseModel):
    """批量操作结果"""
    total: int
    success_count: int
    fail_count: int
    results: List[BatchResultItem]
```

---

## 8. API 测试示例

### 8.1 使用 curl 测试

```bash
# 添加学生
curl -X POST http://localhost:8000/api/v1/students \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "20260001",
    "name": "张三",
    "gender": "男",
    "class_name": "三年一班",
    "enrollment_year": 2026
  }'

# 查询学生列表
curl http://localhost:8000/api/v1/students?class_name=三年一班

# 录入成绩
curl -X POST http://localhost:8000/api/v1/grades \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "20260001",
    "subject": "数学",
    "score": 95.5,
    "exam_type": "期中",
    "exam_date": "2026-04-15"
  }'

# 查询统计数据
curl "http://localhost:8000/api/v1/statistics?class_name=三年一班&subject=数学&metrics=avg,pass_rate"
```

### 8.2 使用 Python requests 测试

```python
import requests

BASE_URL = "http://localhost:8000/api/v1"

# 添加学生
response = requests.post(f"{BASE_URL}/students", json={
    "student_id": "20260001",
    "name": "张三",
    "gender": "男",
    "class_name": "三年一班",
    "enrollment_year": 2026
})
print(response.json())

# 批量录入成绩
response = requests.post(f"{BASE_URL}/grades/batch", json={
    "subject": "数学",
    "exam_type": "期中",
    "exam_date": "2026-04-15",
    "grades": [
        {"student_id": "20260001", "score": 95.5},
        {"student_id": "20260002", "score": 88.0}
    ]
})
print(response.json())

# 获取排名
response = requests.get(f"{BASE_URL}/statistics/ranking", params={
    "class_name": "三年一班",
    "subject": "数学",
    "exam_type": "期中",
    "scope": "class",
    "limit": 10
})
print(response.json())
```

---

## 9. Swagger/OpenAPI 文档

启动服务后，可通过以下地址访问自动生成的 API 文档：

| 地址 | 说明 |
|------|------|
| `http://localhost:8000/docs` | Swagger UI（交互式文档） |
| `http://localhost:8000/redoc` | ReDoc（更美观的文档） |
| `http://localhost:8000/openapi.json` | OpenAPI 规范 JSON |

---

## 附录

### A. 常见问题

**Q: 为什么成绩录入时返回 409 错误？**  
A: 同一学生、同一科目、同一考试类型只能有一条成绩记录。如需修改，请使用 PUT 接口更新。

**Q: 批量录入时部分失败怎么办？**  
A: 响应中会详细列出每条记录的状态，成功的已入库，失败的可根据错误信息修正后重新录入。

**Q: 如何查询某学生的所有成绩？**  
A: 使用 `GET /api/v1/grades?student_id=20260001` 接口。

### B. 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| V1.0 | 2026-06-05 | 初始版本 |

---

> **文档结束**  
> 如有疑问请联系架构团队
