# 学生成绩管理系统 - 系统架构文档

> **文档版本：** V2.0  
> **创建日期：** 2026-06-05  
> **更新日期：** 2026-06-06  
> **架构师：** Architect Agent  
> **文档状态：** 已更新（添加前端架构设计）

---

## 1. 架构概述

### 1.1 架构风格

本系统采用 **分层架构（Layered Architecture）** 结合 **Repository 模式**，将系统划分为四个清晰的层次：

```
┌─────────────────────────────────────────────────────────────┐
│                    表现层 (Presentation Layer)                │
│              CLI 命令行界面 / RESTful API                     │
├─────────────────────────────────────────────────────────────┤
│                    业务逻辑层 (Business Layer)                │
│         StudentService / GradeService / StatisticsService    │
├─────────────────────────────────────────────────────────────┤
│                    数据访问层 (Data Access Layer)             │
│           StudentRepository / GradeRepository                │
├─────────────────────────────────────────────────────────────┤
│                    数据存储层 (Storage Layer)                 │
│                    SQLite 数据库                              │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 架构原则

| 原则 | 说明 | 实践 |
|------|------|------|
| **单一职责** | 每个模块只负责一个功能领域 | Service 层只处理业务逻辑，不直接操作数据库 |
| **依赖倒置** | 高层模块不依赖低层模块 | Service 依赖 Repository 接口，而非具体实现 |
| **接口隔离** | 使用小而专一的接口 | 每个 Repository 接口只包含必要的方法 |
| **开闭原则** | 对扩展开放，对修改关闭 | 新增统计功能只需扩展 Service，无需修改现有代码 |

---

## 2. 技术选型

### 2.1 技术栈总览

| 层次 | 技术选择 | 版本要求 | 选择理由 |
|------|---------|---------|---------|
| **编程语言** | Python | 3.8+ | PRD 要求，生态丰富，开发效率高 |
| **Web 框架** | FastAPI | 0.100+ | 高性能、自动文档生成、类型提示支持 |
| **数据验证** | Pydantic | 2.0+ | 数据校验、序列化、与 FastAPI 深度集成 |
| **数据库** | SQLite | 3.35+ | 轻量级、零配置、适合单机应用、支持 JSON |
| **ORM** | SQLAlchemy | 2.0+ | 成熟稳定、支持多种数据库、类型映射 |
| **CLI 框架** | Typer | 0.9+ | 基于类型提示、自动生成帮助文档 |
| **测试框架** | pytest | 7.0+ | 简洁语法、丰富插件、fixture 机制 |

### 2.2 前端技术选型

| 类别 | 技术选择 | 版本要求 | 选择理由 |
|------|---------|---------|---------|
| **前端框架** | Vue 3 | 3.3+ | 组合式API、TypeScript支持、生态成熟 |
| **UI组件库** | Element Plus | 2.4+ | Vue 3原生支持、组件丰富、文档完善 |
| **图表库** | ECharts | 5.4+ | 功能强大、支持多种图表类型、响应式 |
| **状态管理** | Pinia | 2.1+ | Vue 3官方推荐、TypeScript友好、轻量级 |
| **HTTP客户端** | Axios | 1.4+ | 拦截器支持、请求取消、自动转换JSON |
| **构建工具** | Vite | 4.4+ | 快速冷启动、热更新、现代化构建 |
| **路由管理** | Vue Router | 4.2+ | Vue 3官方路由、支持嵌套路由、导航守卫 |
| **代码规范** | ESLint + Prettier | ESLint 8.44+ / Prettier 3.0+ | 代码格式统一、自动修复 |
| **测试框架** | Vitest | 0.34+ | 与Vite深度集成、速度快、兼容Jest API |

### 2.3 技术选型详细说明

#### 2.3.1 为什么选择 FastAPI 而非 Flask？

| 对比项 | FastAPI | Flask |
|--------|---------|-------|
| 性能 | 基于 Starlette，异步支持，性能接近 Node.js | 同步框架，性能一般 |
| 类型安全 | 原生支持 Pydantic，自动校验 | 需要额外扩展 |
| 文档 | 自动生成 OpenAPI/Swagger 文档 | 需要手动维护 |
| 开发体验 | 类型提示驱动，IDE 支持好 | 灵活但缺乏约束 |

**结论：** FastAPI 更适合需要高性能和类型安全的场景，且自动文档功能可大幅降低 API 维护成本。

#### 2.3.2 为什么选择 SQLite 而非 PostgreSQL/MySQL？

| 对比项 | SQLite | PostgreSQL/MySQL |
|--------|--------|------------------|
| 部署复杂度 | 零配置，单文件数据库 | 需要独立安装和配置 |
| 适用场景 | 单机应用、数据量 < 10 万 | 分布式系统、高并发 |
| 维护成本 | 无需 DBA | 需要专业维护 |
| PRD 匹配度 | 完全满足（10,000 学生） | 过度设计 |

**结论：** PRD 明确为"桌面/命令行工具"，SQLite 是最佳选择。若未来需要扩展为 Web 服务，可无缝切换至 PostgreSQL。

#### 2.3.3 为什么选择 SQLAlchemy 而非直接 SQL？

- **数据库抽象：** 未来切换数据库只需修改连接字符串
- **类型映射：** Python 类与数据库表自动映射
- **迁移支持：** 配合 Alembic 可实现数据库版本管理
- **查询构建：** 避免 SQL 注入，代码更安全

---

## 3. 模块划分

### 3.1 项目目录结构

```
student-grade-system/
├── src/
│   ├── __init__.py
│   ├── main.py                    # 应用入口
│   │
│   ├── api/                       # API 层（表现层）
│   │   ├── __init__.py
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   ├── students.py        # 学生信息 API
│   │   │   ├── grades.py          # 成绩管理 API
│   │   │   └── statistics.py      # 统计分析 API
│   │   └── dependencies.py        # 依赖注入
│   │
│   ├── core/                      # 核心配置
│   │   ├── __init__.py
│   │   ├── config.py              # 应用配置
│   │   ├── database.py            # 数据库连接
│   │   └── exceptions.py          # 自定义异常
│   │
│   ├── models/                    # 数据模型层
│   │   ├── __init__.py
│   │   ├── student.py             # 学生模型
│   │   └── grade.py               # 成绩模型
│   │
│   ├── schemas/                   # Pydantic 模式（数据验证）
│   │   ├── __init__.py
│   │   ├── student.py             # 学生数据模式
│   │   ├── grade.py               # 成绩数据模式
│   │   └── statistics.py          # 统计数据模式
│   │
│   ├── repositories/              # 数据访问层
│   │   ├── __init__.py
│   │   ├── base.py                # 基础 Repository
│   │   ├── student_repo.py        # 学生数据访问
│   │   └── grade_repo.py          # 成绩数据访问
│   │
│   ├── services/                  # 业务逻辑层
│   │   ├── __init__.py
│   │   ├── student_service.py     # 学生业务逻辑
│   │   ├── grade_service.py       # 成绩业务逻辑
│   │   └── statistics_service.py  # 统计业务逻辑
│   │
│   └── cli/                       # CLI 命令行界面
│       ├── __init__.py
│       ├── app.py                 # CLI 应用
│       └── commands/
│           ├── __init__.py
│           ├── student_cmd.py     # 学生相关命令
│           └── grade_cmd.py       # 成绩相关命令
│
├── tests/                         # 测试目录
│   ├── __init__.py
│   ├── conftest.py                # pytest 配置
│   ├── unit/                      # 单元测试
│   │   ├── test_services/
│   │   └── test_repositories/
│   └── integration/               # 集成测试
│       └── test_api/
│
├── docs/                          # 文档目录
│   ├── architecture.md            # 架构文档（本文件）
│   ├── api-spec.md                # API 规范
│   └── prd.md                     # 产品需求文档
│
├── data/                          # 数据目录
│   └── grades.db                  # SQLite 数据库文件
│
├── pyproject.toml                 # 项目配置
├── requirements.txt               # 依赖列表
└── README.md                      # 项目说明
```

### 3.2 模块职责说明

#### 3.2.1 API 层（api/）

**职责：** 处理 HTTP 请求，调用 Service 层，返回响应。

| 模块 | 职责 | 依赖 |
|------|------|------|
| `routes/students.py` | 学生信息 CRUD 接口 | StudentService |
| `routes/grades.py` | 成绩管理接口 | GradeService |
| `routes/statistics.py` | 统计分析接口 | StatisticsService |
| `dependencies.py` | 依赖注入配置 | 数据库连接、Service 实例 |

#### 3.2.2 核心配置层（core/）

**职责：** 应用全局配置、数据库连接、异常定义。

| 模块 | 职责 |
|------|------|
| `config.py` | 环境变量、应用配置（数据库路径、分页大小等） |
| `database.py` | SQLAlchemy 引擎、会话工厂、数据库初始化 |
| `exceptions.py` | 自定义业务异常（StudentNotFound、DuplicateGrade 等） |

#### 3.2.3 数据模型层（models/）

**职责：** 定义 SQLAlchemy ORM 模型，映射数据库表结构。

| 模块 | 模型类 | 数据库表 |
|------|--------|----------|
| `student.py` | Student | students |
| `grade.py` | Grade | grades |

#### 3.2.4 数据验证层（schemas/）

**职责：** 定义 Pydantic 模式，用于请求/响应数据验证和序列化。

| 模块 | 模式类 | 用途 |
|------|--------|------|
| `student.py` | StudentCreate, StudentUpdate, StudentResponse | 学生数据验证 |
| `grade.py` | GradeCreate, GradeBatchCreate, GradeResponse | 成绩数据验证 |
| `statistics.py` | StatisticsQuery, StatisticsResponse | 统计数据验证 |

#### 3.2.5 数据访问层（repositories/）

**职责：** 封装数据库操作，提供数据访问抽象。

| 模块 | 类 | 方法示例 |
|------|-----|---------|
| `base.py` | BaseRepository | create, get_by_id, update, delete |
| `student_repo.py` | StudentRepository | get_by_student_id, get_by_class |
| `grade_repo.py` | GradeRepository | get_by_student, get_by_subject |

#### 3.2.6 业务逻辑层（services/）

**职责：** 实现核心业务逻辑，协调多个 Repository。

| 模块 | 类 | 核心功能 |
|------|-----|---------|
| `student_service.py` | StudentService | 学生增删改查、学号唯一性校验 |
| `grade_service.py` | GradeService | 成绩录入、批量导入、成绩修改 |
| `statistics_service.py` | StatisticsService | 平均分、及格率、排名计算 |

#### 3.2.7 CLI 命令行层（cli/）

**职责：** 提供命令行交互界面。

| 模块 | 命令示例 |
|------|---------|
| `student_cmd.py` | `student add`, `student list`, `student search` |
| `grade_cmd.py` | `grade input`, `grade batch`, `grade query` |

#### 3.3 前端模块划分

**职责：** 提供Web前端界面，实现用户交互和数据展示。

##### 3.3.1 前端目录结构

```
frontend/
├── public/                    # 静态资源
│   ├── favicon.ico           # 网站图标
│   └── index.html            # HTML入口模板
├── src/
│   ├── api/                  # API接口封装
│   │   ├── index.ts          # Axios实例配置
│   │   ├── student.ts        # 学生相关API
│   │   ├── grade.ts          # 成绩相关API
│   │   └── statistics.ts     # 统计相关API
│   ├── assets/               # 静态资源
│   │   ├── images/           # 图片资源
│   │   └── styles/           # 全局样式
│   ├── components/           # 公共组件
│   │   ├── layout/           # 布局组件
│   │   │   ├── AppHeader.vue # 头部导航
│   │   │   ├── AppSidebar.vue# 侧边栏
│   │   │   └── AppLayout.vue # 整体布局
│   │   ├── common/           # 通用组件
│   │   │   ├── DataTable.vue # 数据表格
│   │   │   ├── SearchForm.vue# 搜索表单
│   │   │   ├── Pagination.vue# 分页组件
│   │   │   └── ConfirmDialog.vue # 确认对话框
│   │   └── chart/            # 图表组件
│   │       ├── BarChart.vue  # 柱状图
│   │       ├── LineChart.vue # 折线图
│   │       ├── PieChart.vue  # 饼图
│   │       └── RadarChart.vue# 雷达图
│   ├── composables/          # 组合式函数
│   │   ├── useStudent.ts     # 学生相关逻辑
│   │   ├── useGrade.ts       # 成绩相关逻辑
│   │   ├── useStatistics.ts  # 统计相关逻辑
│   │   └── usePagination.ts  # 分页逻辑
│   ├── router/               # 路由配置
│   │   └── index.ts          # 路由定义
│   ├── stores/               # 状态管理
│   │   ├── student.ts        # 学生状态
│   │   ├── grade.ts          # 成绩状态
│   │   └── app.ts            # 应用全局状态
│   ├── types/                # TypeScript类型定义
│   │   ├── student.ts        # 学生类型
│   │   ├── grade.ts          # 成绩类型
│   │   └── statistics.ts     # 统计类型
│   ├── utils/                # 工具函数
│   │   ├── request.ts        # 请求工具
│   │   ├── format.ts         # 格式化工具
│   │   └── validation.ts     # 验证工具
│   ├── views/                # 页面组件
│   │   ├── student/          # 学生管理页面
│   │   │   ├── StudentList.vue   # 学生列表
│   │   │   ├── StudentForm.vue   # 学生表单
│   │   │   └── StudentDetail.vue # 学生详情
│   │   ├── grade/            # 成绩管理页面
│   │   │   ├── GradeList.vue     # 成绩列表
│   │   │   ├── GradeForm.vue     # 成绩录入
│   │   │   └── GradeImport.vue   # 成绩导入
│   │   ├── statistics/       # 统计分析页面
│   │   │   ├── StatisticsOverview.vue # 统计概览
│   │   │   ├── ClassStatistics.vue    # 班级统计
│   │   │   └── SubjectStatistics.vue  # 科目统计
│   │   └── dashboard/        # 仪表盘
│   │       └── Dashboard.vue     # 主仪表盘
│   ├── App.vue               # 根组件
│   └── main.ts               # 应用入口
├── package.json              # 依赖配置
├── vite.config.ts            # Vite配置
├── tsconfig.json             # TypeScript配置
├── .eslintrc.cjs             # ESLint配置
└── .prettierrc               # Prettier配置
```

##### 3.3.2 前端模块职责说明

| 模块 | 职责 | 主要文件 |
|------|------|----------|
| **api/** | 封装后端API调用，统一请求处理 | student.ts, grade.ts, statistics.ts |
| **components/** | 可复用的UI组件，不包含业务逻辑 | DataTable.vue, SearchForm.vue, Chart组件 |
| **composables/** | 组合式函数，封装业务逻辑和状态 | useStudent.ts, useGrade.ts |
| **router/** | 前端路由配置，页面导航控制 | index.ts |
| **stores/** | Pinia状态管理，全局状态存储 | student.ts, grade.ts, app.ts |
| **types/** | TypeScript类型定义，确保类型安全 | student.ts, grade.ts |
| **utils/** | 工具函数，通用功能封装 | request.ts, format.ts |
| **views/** | 页面组件，包含具体业务逻辑 | StudentList.vue, GradeForm.vue |

---

## 4. 前端架构设计

### 4.1 前端整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                    用户界面层 (View Layer)                    │
│              Vue 3 组件 + Element Plus UI                    │
├─────────────────────────────────────────────────────────────┤
│                    状态管理层 (State Layer)                   │
│                    Pinia Store                               │
├─────────────────────────────────────────────────────────────┤
│                    业务逻辑层 (Logic Layer)                   │
│              Composables 组合式函数                          │
├─────────────────────────────────────────────────────────────┤
│                    数据访问层 (API Layer)                     │
│              Axios HTTP 客户端                               │
├─────────────────────────────────────────────────────────────┤
│                    后端服务层 (Backend Layer)                 │
│              FastAPI RESTful API                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 前端与后端接口对接方案

#### 4.2.1 API 请求封装

```typescript
// src/api/index.ts
import axios from 'axios'
import type { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios'
import { ElMessage } from 'element-plus'

// 创建Axios实例
const service: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || '/api/v1',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// 请求拦截器
service.interceptors.request.use(
  (config) => {
    // 可以在这里添加token等认证信息
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

// 响应拦截器
service.interceptors.response.use(
  (response: AxiosResponse) => {
    const { data } = response
    
    // 根据后端返回的数据结构处理
    if (data.success === false) {
      ElMessage.error(data.error?.message || '请求失败')
      return Promise.reject(new Error(data.error?.message || '请求失败'))
    }
    
    return data
  },
  (error) => {
    // 处理HTTP错误状态码
    const message = error.response?.data?.error?.message || error.message || '网络错误'
    ElMessage.error(message)
    return Promise.reject(error)
  }
)

export default service
```

#### 4.2.2 API 接口定义示例

```typescript
// src/api/student.ts
import request from './index'
import type { Student, StudentCreate, StudentUpdate, StudentListResponse } from '@/types/student'

// 获取学生列表
export function getStudentList(params: {
  page?: number
  page_size?: number
  class_name?: string
  keyword?: string
}) {
  return request.get<any, StudentListResponse>('/students', { params })
}

// 获取学生详情
export function getStudentDetail(studentId: string) {
  return request.get<any, Student>(`/students/${studentId}`)
}

// 创建学生
export function createStudent(data: StudentCreate) {
  return request.post<any, Student>('/students', data)
}

// 更新学生信息
export function updateStudent(studentId: string, data: StudentUpdate) {
  return request.put<any, Student>(`/students/${studentId}`, data)
}

// 删除学生
export function deleteStudent(studentId: string) {
  return request.delete<any, void>(`/students/${studentId}`)
}
```

#### 4.2.3 接口对接规范

| 规范项 | 说明 | 示例 |
|--------|------|------|
| **基础路径** | 所有API以`/api/v1`为前缀 | `/api/v1/students` |
| **请求方法** | 遵循RESTful规范 | GET(查询)、POST(创建)、PUT(更新)、DELETE(删除) |
| **响应格式** | 统一JSON格式 | `{ success: true, data: {}, error: null }` |
| **分页参数** | 使用`page`和`page_size` | `?page=1&page_size=20` |
| **错误处理** | HTTP状态码 + 业务错误码 | 404 + `STUDENT_NOT_FOUND` |
| **数据验证** | 前端表单验证 + 后端Pydantic验证 | 双重验证确保数据安全 |

### 4.3 前端状态管理方案

#### 4.3.1 Pinia Store 设计

```typescript
// src/stores/student.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { Student, StudentCreate, StudentUpdate } from '@/types/student'
import * as studentApi from '@/api/student'

export const useStudentStore = defineStore('student', () => {
  // 状态
  const students = ref<Student[]>([])
  const currentStudent = ref<Student | null>(null)
  const loading = ref(false)
  const pagination = ref({
    page: 1,
    pageSize: 20,
    total: 0
  })

  // 计算属性
  const studentCount = computed(() => pagination.value.total)
  const hasStudents = computed(() => students.value.length > 0)

  // 操作
  async function fetchStudents(params?: {
    class_name?: string
    keyword?: string
  }) {
    loading.value = true
    try {
      const response = await studentApi.getStudentList({
        page: pagination.value.page,
        page_size: pagination.value.pageSize,
        ...params
      })
      students.value = response.items
      pagination.value.total = response.total
    } catch (error) {
      console.error('获取学生列表失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  async function fetchStudentDetail(studentId: string) {
    loading.value = true
    try {
      currentStudent.value = await studentApi.getStudentDetail(studentId)
    } catch (error) {
      console.error('获取学生详情失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  async function createStudent(data: StudentCreate) {
    loading.value = true
    try {
      const newStudent = await studentApi.createStudent(data)
      students.value.unshift(newStudent)
      pagination.value.total++
      return newStudent
    } catch (error) {
      console.error('创建学生失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  async function updateStudent(studentId: string, data: StudentUpdate) {
    loading.value = true
    try {
      const updatedStudent = await studentApi.updateStudent(studentId, data)
      const index = students.value.findIndex(s => s.student_id === studentId)
      if (index !== -1) {
        students.value[index] = updatedStudent
      }
      if (currentStudent.value?.student_id === studentId) {
        currentStudent.value = updatedStudent
      }
      return updatedStudent
    } catch (error) {
      console.error('更新学生失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  async function deleteStudent(studentId: string) {
    loading.value = true
    try {
      await studentApi.deleteStudent(studentId)
      students.value = students.value.filter(s => s.student_id !== studentId)
      pagination.value.total--
      if (currentStudent.value?.student_id === studentId) {
        currentStudent.value = null
      }
    } catch (error) {
      console.error('删除学生失败:', error)
      throw error
    } finally {
      loading.value = false
    }
  }

  function setPage(page: number) {
    pagination.value.page = page
  }

  function setPageSize(pageSize: number) {
    pagination.value.pageSize = pageSize
    pagination.value.page = 1
  }

  return {
    // 状态
    students,
    currentStudent,
    loading,
    pagination,
    
    // 计算属性
    studentCount,
    hasStudents,
    
    // 操作
    fetchStudents,
    fetchStudentDetail,
    createStudent,
    updateStudent,
    deleteStudent,
    setPage,
    setPageSize
  }
})
```

#### 4.3.2 状态管理规范

| 规范项 | 说明 | 示例 |
|--------|------|------|
| **Store 拆分** | 按业务模块拆分Store | `useStudentStore`、`useGradeStore` |
| **状态定义** | 使用`ref()`定义响应式状态 | `const students = ref<Student[]>([])` |
| **计算属性** | 使用`computed()`派生状态 | `const studentCount = computed(() => ...)` |
| **异步操作** | 使用`async/await`处理API调用 | `async function fetchStudents() {...}` |
| **错误处理** | 统一捕获并处理错误 | `try {...} catch (error) {...}` |
| **加载状态** | 每个Store维护`loading`状态 | `const loading = ref(false)` |

### 4.4 前端路由设计

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    redirect: '/dashboard'
  },
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('@/views/dashboard/Dashboard.vue'),
    meta: { title: '仪表盘', icon: 'Dashboard' }
  },
  {
    path: '/student',
    name: 'Student',
    redirect: '/student/list',
    children: [
      {
        path: 'list',
        name: 'StudentList',
        component: () => import('@/views/student/StudentList.vue'),
        meta: { title: '学生列表', icon: 'User' }
      },
      {
        path: 'add',
        name: 'StudentAdd',
        component: () => import('@/views/student/StudentForm.vue'),
        meta: { title: '添加学生', icon: 'UserPlus' }
      },
      {
        path: 'edit/:id',
        name: 'StudentEdit',
        component: () => import('@/views/student/StudentForm.vue'),
        meta: { title: '编辑学生', icon: 'UserEdit', hidden: true }
      },
      {
        path: 'detail/:id',
        name: 'StudentDetail',
        component: () => import('@/views/student/StudentDetail.vue'),
        meta: { title: '学生详情', icon: 'User', hidden: true }
      }
    ]
  },
  {
    path: '/grade',
    name: 'Grade',
    redirect: '/grade/list',
    children: [
      {
        path: 'list',
        name: 'GradeList',
        component: () => import('@/views/grade/GradeList.vue'),
        meta: { title: '成绩列表', icon: 'Document' }
      },
      {
        path: 'input',
        name: 'GradeInput',
        component: () => import('@/views/grade/GradeForm.vue'),
        meta: { title: '成绩录入', icon: 'Edit' }
      },
      {
        path: 'import',
        name: 'GradeImport',
        component: () => import('@/views/grade/GradeImport.vue'),
        meta: { title: '成绩导入', icon: 'Upload' }
      }
    ]
  },
  {
    path: '/statistics',
    name: 'Statistics',
    redirect: '/statistics/overview',
    children: [
      {
        path: 'overview',
        name: 'StatisticsOverview',
        component: () => import('@/views/statistics/StatisticsOverview.vue'),
        meta: { title: '统计概览', icon: 'DataAnalysis' }
      },
      {
        path: 'class',
        name: 'ClassStatistics',
        component: () => import('@/views/statistics/ClassStatistics.vue'),
        meta: { title: '班级统计', icon: 'School' }
      },
      {
        path: 'subject',
        name: 'SubjectStatistics',
        component: () => import('@/views/statistics/SubjectStatistics.vue'),
        meta: { title: '科目统计', icon: 'Collection' }
      }
    ]
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 路由守卫
router.beforeEach((to, from, next) => {
  // 设置页面标题
  document.title = `${to.meta.title || '学生成绩管理系统'} - 学生成绩管理系统`
  next()
})

export default router
```

### 4.5 前端组件设计原则

#### 4.5.1 组件分类

| 组件类型 | 说明 | 示例 |
|----------|------|------|
| **布局组件** | 页面整体布局结构 | AppHeader、AppSidebar、AppLayout |
| **通用组件** | 无业务逻辑的可复用组件 | DataTable、SearchForm、Pagination |
| **业务组件** | 包含特定业务逻辑的组件 | StudentForm、GradeTable |
| **页面组件** | 完整的页面视图 | StudentList、GradeForm |

#### 4.5.2 组件设计规范

1. **单一职责原则**：每个组件只负责一个功能
2. **Props 向下，Events 向上**：通过Props传递数据，通过Events触发操作
3. **组合式函数复用**：使用composables封装可复用逻辑
4. **TypeScript 类型**：为Props和Emits定义明确的类型
5. **响应式设计**：使用CSS媒体查询或组件库的响应式特性

### 4.6 前端性能优化策略

| 优化项 | 策略 | 说明 |
|--------|------|------|
| **代码分割** | 路由懒加载 | `component: () => import(...)` |
| **组件按需引入** | Element Plus 按需导入 | 使用unplugin-vue-components |
| **图片优化** | 图片压缩、懒加载 | 使用v-lazy指令 |
| **缓存策略** | API响应缓存 | 使用Pinia持久化或localStorage |
| **虚拟滚动** | 大数据量表格 | 使用虚拟滚动组件 |
| **防抖节流** | 搜索、滚动等操作 | 使用lodash工具函数 |

---

## 5. 数据模型设计

### 5.1 ER 图（实体关系图）

```
┌─────────────────────────────────────────────────────────────┐
│                        students 表                           │
├─────────────────────────────────────────────────────────────┤
│ PK │ student_id    VARCHAR(8)    学号（如：20260001）          │
├────┼─────────────────────────────────────────────────────────┤
│    │ name          VARCHAR(20)   姓名                        │
│    │ gender        VARCHAR(2)    性别（男/女）                │
│    │ class_name    VARCHAR(20)   班级（如：三年一班）          │
│    │ enrollment_year INTEGER     入学年份                    │
│    │ created_at    DATETIME      创建时间                    │
│    │ updated_at    DATETIME      更新时间                    │
└────┴─────────────────────────────────────────────────────────┘
                              │
                              │ 1:N（一对多）
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                        grades 表                             │
├─────────────────────────────────────────────────────────────┤
│ PK │ grade_id      INTEGER       成绩ID（自增主键）           │
├────┼─────────────────────────────────────────────────────────┤
│ FK │ student_id    VARCHAR(8)    学号（外键关联 students 表）  │
├────┼─────────────────────────────────────────────────────────┤
│    │ subject       VARCHAR(10)   科目                        │
│    │ score         DECIMAL(4,1)  分数（0-100，支持1位小数）    │
│    │ exam_type     VARCHAR(10)   考试类型                    │
│    │ exam_date     DATE          考试日期                    │
│    │ created_at    DATETIME      创建时间                    │
│    │ updated_at    DATETIME      更新时间                    │
└────┴─────────────────────────────────────────────────────────┘

索引设计：
- students 表：student_id（主键）、class_name（普通索引）
- grades 表：grade_id（主键）、student_id（普通索引）、
            (student_id, subject, exam_type)（唯一复合索引）
```

### 5.2 SQLAlchemy 模型定义

#### 5.2.1 学生模型

```python
# src/models/student.py
from datetime import datetime
from sqlalchemy import Column, String, Integer, DateTime
from sqlalchemy.orm import relationship
from src.core.database import Base

class Student(Base):
    """学生信息模型"""
    __tablename__ = "students"

    # 主键：学号（格式：YYYY + 4位序号，如 20260001）
    student_id = Column(String(8), primary_key=True, index=True)
    
    # 姓名：2-20个字符
    name = Column(String(20), nullable=False)
    
    # 性别：男/女
    gender = Column(String(2), nullable=False)
    
    # 班级：格式如 "三年一班"
    class_name = Column(String(20), nullable=False, index=True)
    
    # 入学年份
    enrollment_year = Column(Integer, nullable=False)
    
    # 时间戳
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # 关联：一个学生有多条成绩记录
    grades = relationship("Grade", back_populates="student", cascade="all, delete-orphan")

    def __repr__(self):
        return f"<Student(student_id='{self.student_id}', name='{self.name}')>"
```

#### 5.2.2 成绩模型

```python
# src/models/grade.py
from datetime import datetime, date
from sqlalchemy import Column, String, Integer, Float, Date, DateTime, ForeignKey, UniqueConstraint
from sqlalchemy.orm import relationship
from src.core.database import Base

class Grade(Base):
    """成绩信息模型"""
    __tablename__ = "grades"

    # 主键：成绩ID（自增）
    grade_id = Column(Integer, primary_key=True, autoincrement=True)
    
    # 外键：学号
    student_id = Column(String(8), ForeignKey("students.student_id"), nullable=False, index=True)
    
    # 科目：语文、数学、英语等
    subject = Column(String(10), nullable=False)
    
    # 分数：0-100，支持1位小数
    score = Column(Float, nullable=False)
    
    # 考试类型：期中、期末、月考、单元测试
    exam_type = Column(String(10), nullable=False)
    
    # 考试日期
    exam_date = Column(Date, nullable=False)
    
    # 时间戳
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # 关联：成绩属于某个学生
    student = relationship("Student", back_populates="grades")

    # 唯一约束：同一学生、同一科目、同一考试类型只能有一条记录
    __table_args__ = (
        UniqueConstraint('student_id', 'subject', 'exam_type', name='uq_student_subject_exam'),
    )

    def __repr__(self):
        return f"<Grade(student_id='{self.student_id}', subject='{self.subject}', score={self.score})>"
```

### 5.3 枚举值定义

```python
# src/core/constants.py

# 科目列表
SUBJECTS = [
    "语文", "数学", "英语",
    "物理", "化学", "生物",
    "历史", "地理", "政治"
]

# 考试类型
EXAM_TYPES = ["期中", "期末", "月考", "单元测试"]

# 性别选项
GENDERS = ["男", "女"]

# 分数范围
SCORE_MIN = 0.0
SCORE_MAX = 100.0

# 学号格式：4位年份 + 4位序号
STUDENT_ID_PATTERN = r"^\d{4}\d{4}$"
```

---

## 6. 核心流程设计

### 6.1 成绩录入流程

```
用户输入成绩数据
       │
       ▼
┌──────────────────┐
│  API 层接收请求   │
│  Pydantic 校验   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Service 层处理   │
│  - 检查学生存在   │
│  - 检查重复记录   │
│  - 校验分数范围   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Repository 层   │
│  - 插入数据库     │
└────────┬─────────┘
         │
         ▼
    返回成功响应
```

### 6.2 统计分析流程

```
用户请求统计（如：班级平均分）
         │
         ▼
┌──────────────────────┐
│  StatisticsService    │
│  - 解析查询条件       │
│  - 调用 GradeRepo     │
│  - 执行聚合查询       │
│  - 计算统计指标       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  返回统计结果         │
│  - 平均分             │
│  - 最高/最低分        │
│  - 及格率/优秀率      │
└──────────────────────┘
```

---

## 7. 错误处理策略

### 7.1 异常层次结构

```python
# src/core/exceptions.py

class AppException(Exception):
    """应用基础异常"""
    def __init__(self, message: str, code: str):
        self.message = message
        self.code = code
        super().__init__(self.message)

class NotFoundException(AppException):
    """资源不存在"""
    def __init__(self, resource: str, identifier: str):
        super().__init__(
            message=f"{resource} '{identifier}' 不存在",
            code="NOT_FOUND"
        )

class DuplicateException(AppException):
    """重复数据异常"""
    def __init__(self, resource: str, field: str, value: str):
        super().__init__(
            message=f"{resource} 的 {field} '{value}' 已存在",
            code="DUPLICATE"
        )

class ValidationException(AppException):
    """数据验证异常"""
    def __init__(self, message: str):
        super().__init__(message=message, code="VALIDATION_ERROR")

# 具体异常类
class StudentNotFoundException(NotFoundException):
    def __init__(self, student_id: str):
        super().__init__("学生", student_id)

class DuplicateGradeException(DuplicateException):
    def __init__(self, student_id: str, subject: str, exam_type: str):
        super().__init__("成绩", "学生-科目-考试类型", f"{student_id}-{subject}-{exam_type}")

class ScoreOutOfRangeException(ValidationException):
    def __init__(self, score: float):
        super().__init__(f"分数 {score} 超出范围（0-100）")
```

### 7.2 全局异常处理

```python
# src/api/exception_handlers.py
from fastapi import Request
from fastapi.responses import JSONResponse
from src.core.exceptions import AppException

async def app_exception_handler(request: Request, exc: AppException):
    """全局异常处理器"""
    status_codes = {
        "NOT_FOUND": 404,
        "DUPLICATE": 409,
        "VALIDATION_ERROR": 422
    }
    
    return JSONResponse(
        status_code=status_codes.get(exc.code, 500),
        content={
            "success": False,
            "error": {
                "code": exc.code,
                "message": exc.message
            }
        }
    )
```

---

## 8. 性能优化策略

### 8.1 数据库优化

| 优化项 | 策略 | 说明 |
|--------|------|------|
| **索引设计** | 为常用查询字段创建索引 | student_id、class_name、subject |
| **复合索引** | 为组合查询创建联合索引 | (student_id, subject, exam_type) |
| **批量操作** | 使用 bulk_insert_mappings | 批量录入时减少数据库交互 |
| **连接池** | 配置 SQLAlchemy 连接池 | pool_size=5, max_overflow=10 |

### 8.2 查询优化

```python
# 使用 SQLAlchemy 的批量操作
from sqlalchemy.dialects.sqlite import insert

def batch_create_grades(grades_data: list[dict]):
    """批量插入成绩记录"""
    stmt = insert(Grade.__table__).values(grades_data)
    stmt = stmt.on_conflict_do_update(
        index_elements=['student_id', 'subject', 'exam_type'],
        set_={'score': stmt.excluded.score}
    )
    session.execute(stmt)
    session.commit()
```

### 8.3 性能指标

| 场景 | 目标 | 实现方式 |
|------|------|---------|
| 单条查询 | < 10ms | 索引优化 |
| 统计计算 | < 100ms | 数据库聚合函数 |
| 批量导入 500 条 | < 1 分钟 | 批量插入 + 事务 |
| 10,000 学生查询 | < 100ms | 分页 + 索引 |

---

## 9. 安全设计

### 9.1 输入验证

```python
# src/schemas/student.py
from pydantic import BaseModel, Field, field_validator
import re

class StudentCreate(BaseModel):
    """学生创建请求模式"""
    student_id: str = Field(..., pattern=r"^\d{8}$", description="学号（8位数字）")
    name: str = Field(..., min_length=2, max_length=20, description="姓名")
    gender: str = Field(..., description="性别")
    class_name: str = Field(..., min_length=2, max_length=20, description="班级")
    enrollment_year: int = Field(..., ge=2000, le=2100, description="入学年份")

    @field_validator('student_id')
    @classmethod
    def validate_student_id(cls, v):
        """验证学号格式：4位年份 + 4位序号"""
        if not re.match(r"^\d{4}\d{4}$", v):
            raise ValueError("学号格式错误，应为8位数字（如：20260001）")
        year = int(v[:4])
        if year < 2000 or year > 2100:
            raise ValueError("学号年份部分应在2000-2100之间")
        return v

    @field_validator('gender')
    @classmethod
    def validate_gender(cls, v):
        if v not in ["男", "女"]:
            raise ValueError("性别只能是 '男' 或 '女'")
        return v
```

### 9.2 SQL 注入防护

- **使用 ORM：** SQLAlchemy 自动参数化查询，防止 SQL 注入
- **输入验证：** Pydantic 严格校验所有输入
- **最小权限：** 数据库用户只授予必要的 CRUD 权限

---

## 10. 可扩展性设计

### 10.1 未来扩展点

| 扩展方向 | 当前设计 | 扩展方案 |
|---------|---------|---------|
| **多数据库支持** | SQLAlchemy ORM | 修改 connection string 即可切换 |
| **Web 前端** | FastAPI 后端 | 添加 Vue/React 前端，API 已就绪 |
| **用户认证** | 无认证 | 添加 JWT 认证中间件 |
| **数据导出** | 无 | 添加 CSV/Excel 导出 Service |
| **缓存** | 无 | 添加 Redis 缓存层 |

### 10.2 插件化设计

```python
# 统计功能可通过插件扩展
class StatisticsPlugin:
    """统计插件基类"""
    def calculate(self, data: list) -> dict:
        raise NotImplementedError

class AveragePlugin(StatisticsPlugin):
    """平均分计算插件"""
    def calculate(self, data: list) -> dict:
        return {"average": sum(data) / len(data)}

class PassRatePlugin(StatisticsPlugin):
    """及格率计算插件"""
    def calculate(self, data: list) -> dict:
        passed = sum(1 for s in data if s >= 60)
        return {"pass_rate": passed / len(data) * 100}
```

---

## 11. 部署架构

### 11.1 单机部署

```
┌─────────────────────────────────────────────┐
│              用户机器                         │
│  ┌───────────────────────────────────────┐  │
│  │         Python 应用                    │  │
│  │  ┌─────────┐  ┌─────────┐  ┌───────┐ │  │
│  │  │ FastAPI  │  │  CLI    │  │ SQLite│ │  │
│  │  │  :8000   │  │         │  │  .db  │ │  │
│  │  └─────────┘  └─────────┘  └───────┘ │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### 11.2 启动方式

```bash
# 方式1：启动 API 服务
uvicorn src.main:app --host 0.0.0.0 --port 8000

# 方式2：使用 CLI
python -m src.cli student list
python -m src.cli grade input --student-id 20260001 --subject 数学 --score 95
```

---

## 附录

### A. 依赖列表

```txt
# requirements.txt
fastapi>=0.100.0
uvicorn>=0.23.0
sqlalchemy>=2.0.0
pydantic>=2.0.0
typer>=0.9.0
python-dateutil>=2.8.0
pytest>=7.0.0
httpx>=0.24.0
```

### B. 前端依赖列表

```json
{
  "dependencies": {
    "vue": "^3.3.0",
    "vue-router": "^4.2.0",
    "pinia": "^2.1.0",
    "axios": "^1.4.0",
    "element-plus": "^2.4.0",
    "echarts": "^5.4.0",
    "@element-plus/icons-vue": "^2.1.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.4.0",
    "vite": "^4.4.0",
    "typescript": "^5.2.0",
    "eslint": "^8.44.0",
    "prettier": "^3.0.0",
    "vitest": "^0.34.0",
    "@vue/test-utils": "^2.4.0",
    "jsdom": "^22.1.0",
    "unplugin-vue-components": "^0.25.0",
    "unplugin-auto-import": "^0.16.0"
  }
}
```

### C. 配置文件示例

```python
# src/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """应用配置"""
    # 数据库配置
    DATABASE_URL: str = "sqlite:///./data/grades.db"
    
    # 应用配置
    APP_NAME: str = "学生成绩管理系统"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False
    
    # 分页配置
    DEFAULT_PAGE_SIZE: int = 20
    MAX_PAGE_SIZE: int = 100
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

---

> **文档结束**  
> 架构设计已完成，详细 API 规范请参考 `api-spec.md`  
> 前端架构设计已添加，包含前端技术选型、模块划分、接口对接和状态管理方案
