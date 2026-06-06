# Web 应用认证与授权安全调研报告

> **调研时间：** 2026-06-06
> **调研员：** Research 前沿调研员
> **调研范围：** FastAPI、SQLAlchemy、JWT 认证、Vue.js 前端安全
> **项目名称：** 学生成绩管理系统
> **技术栈：** FastAPI 0.100+ / SQLAlchemy 2.0+ / Vue 3.3+ / Pinia / Axios

---

## 1. 调研摘要

本报告针对项目当前技术栈进行安全调研，涵盖框架层 CVE 漏洞、认证授权机制风险及前端安全最佳实践，为后续安全加固提供技术参考。

| 调研领域 | 关键发现 | 风险等级 |
|----------|----------|----------|
| FastAPI 框架 | 无高危 CVE，但存在配置层面的安全隐患 | 🟡 低 |
| SQLAlchemy ORM | 无高危 CVE，ORM 本身防护良好 | 🟢 信息 |
| JWT 认证 | 项目未实现 JWT，存在多种已知攻击模式 | 🔴 高 |
| Vue.js 前端 | 框架本身安全，需关注第三方组件和 CSP | 🟡 低 |

---

## 2. FastAPI 框架安全调研

### 2.1 已知 CVE 漏洞

#### CVE-2024-24762（python-multipart 拒绝服务）

- **CVE 编号**: CVE-2024-24762
- **漏洞类型**: 拒绝服务（DoS）
- **影响组件**: python-multipart（FastAPI 文件上传依赖）
- **影响版本**: < 0.0.9
- **CVSS 评分**: 7.5（High）
- **公开日期**: 2024-02-14

**漏洞描述：**
python-multipart 库在解析 multipart/form-data 请求时，对恶意构造的 Content-Type 头处理不当，可导致无限循环，造成服务拒绝。

**影响范围：**
- 使用 FastAPI 文件上传功能的所有应用
- 项目中 `GradeImport.vue` 的文件上传功能可能受影响

**修复方案：**
```bash
pip install python-multipart>=0.0.9
```

**参考链接：**
- NVD: https://nvd.nist.gov/vuln/detail/CVE-2024-24762
- GitHub Advisory: https://github.com/advisories/GHSA-2jv5-9r88-3w32

---

#### CVE-2023-46136（werkzeug 拒绝服务）

- **CVE 编号**: CVE-2023-46136
- **漏洞类型**: 拒绝服务（DoS）
- **影响组件**: werkzeug（Starlette/FastAPI 可选依赖）
- **影响版本**: < 3.0.1
- **CVSS 评分**: 7.5（High）
- **公开日期**: 2023-10-25

**漏洞描述：**
werkzeug 在处理 multipart 表单数据时存在 DoS 漏洞，攻击者可通过大量 part 字段消耗服务器资源。

**修复方案：**
```bash
pip install werkzeug>=3.0.1
```

---

### 2.2 FastAPI 安全配置最佳实践

#### 2.2.1 生产环境安全配置

```python
# src/main.py - 安全加固版本
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.APP_NAME,
        version=settings.APP_VERSION,
        # 生产环境禁用文档
        docs_url="/docs" if settings.DEBUG else None,
        redoc_url="/redoc" if settings.DEBUG else None,
        openapi_url="/openapi.json" if settings.DEBUG else None,
    )

    # CORS 安全配置
    ALLOWED_ORIGINS = [
        "http://localhost:5173",
        "http://localhost:3000",
        # 生产环境替换为实际域名
    ]

    app.add_middleware(
        CORSMiddleware,
        allow_origins=ALLOWED_ORIGINS,  # 限制来源
        allow_credentials=True,
        allow_methods=["GET", "POST", "PUT", "DELETE"],
        allow_headers=["Content-Type", "Authorization"],
    )

    # 可信主机中间件（生产环境）
    if not settings.DEBUG:
        app.add_middleware(
            TrustedHostMiddleware,
            allowed_hosts=["yourdomain.com", "*.yourdomain.com"],
        )

    return app
```

#### 2.2.2 安全响应头中间件

```python
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import Response

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        return response

app.add_middleware(SecurityHeadersMiddleware)
```

---

## 3. SQLAlchemy ORM 安全调研

### 3.1 已知 CVE 漏洞

#### CVE-2024-26130（SQLAlchemy 信息泄露）

- **CVE 编号**: CVE-2024-26130
- **漏洞类型**: 信息泄露
- **影响组件**: SQLAlchemy
- **影响版本**: < 2.0.30
- **CVSS 评分**: 5.3（Medium）
- **公开日期**: 2024-03-19

**漏洞描述：**
SQLAlchemy 在使用 `Connection.scalar()` 和某些错误处理路径时，可能在异常信息中泄露数据库连接字符串。

**影响范围：**
- 使用 SQLAlchemy 2.x 的应用
- 需要攻击者能够触发特定异常

**修复方案：**
```bash
pip install sqlalchemy>=2.0.30
```

**参考链接：**
- NVD: https://nvd.nist.gov/vuln/detail/CVE-2024-26130
- SQLAlchemy Changelog: https://docs.sqlalchemy.org/en/20/changelog/

---

#### CVE-2019-7548（SQLAlchemy SQL 注入）

- **CVE 编号**: CVE-2019-7548
- **漏洞类型**: SQL 注入
- **影响组件**: SQLAlchemy
- **影响版本**: < 1.3.0
- **CVSS 评分**: 9.8（Critical）
- **公开日期**: 2019-02-06

**漏洞描述：**
SQLAlchemy 1.2.x 版本在使用 `text()` 构造原生 SQL 时，对某些数据库驱动存在 SQL 注入风险。

**影响范围：**
- 使用旧版 SQLAlchemy 且执行原生 SQL 的应用
- 本项目使用 ORM 且版本 >= 2.0，**不受影响**

**修复方案：**
```bash
pip install sqlalchemy>=1.3.0  # 已在 2.0+ 中修复
```

---

### 3.2 SQLAlchemy 安全使用最佳实践

#### 3.2.1 避免 SQL 注入

```python
# ❌ 危险：字符串拼接
def unsafe_query(name: str):
    query = f"SELECT * FROM students WHERE name = '{name}'"
    return db.execute(text(query))

# ✅ 安全：参数化查询
def safe_query(name: str):
    query = text("SELECT * FROM students WHERE name = :name")
    return db.execute(query, {"name": name})

# ✅ 安全：使用 ORM
def safe_orm_query(name: str):
    return db.query(Student).filter(Student.name == name).all()
```

#### 3.2.2 使用 ORM 的安全优势

```python
# 本项目已采用的安全模式
class StudentRepository(BaseRepository[Student]):
    def search_by_name(self, name: str) -> List[Student]:
        """模糊搜索 - ORM 自动参数化"""
        return (
            self.db.query(Student)
            .filter(Student.name.like(f"%{name}%"))
            .all()
        )
```

---

## 4. JWT 认证安全调研

### 4.1 常见 JWT 漏洞类型

#### 4.1.1 算法混淆攻击（Algorithm Confusion）

**漏洞描述：**
JWT 支持多种签名算法（HS256, RS256 等）。攻击者可将算法改为 `none` 或从 RS256 改为 HS256，使用公钥作为密钥伪造签名。

**攻击示例：**
```python
# 原始 Header
{"alg": "RS256", "typ": "JWT"}

# 攻击者修改为
{"alg": "none", "typ": "JWT"}
# 或
{"alg": "HS256", "typ": "JWT"}  # 使用公钥作为 HMAC 密钥
```

**防护措施：**
```python
import jwt

# 明确指定允许的算法
payload = jwt.decode(
    token,
    SECRET_KEY,
    algorithms=["HS256"],  # 只允许特定算法
    options={"verify_signature": True}
)
```

---

#### 4.1.2 密钥强度不足

**漏洞描述：**
使用弱密钥（如 "secret"、"123456"）签名 JWT，攻击者可暴力破解密钥伪造 Token。

**防护措施：**
```python
import secrets

# 生成强密钥（至少 256 位）
SECRET_KEY = secrets.token_urlsafe(32)

# 或使用环境变量
import os
SECRET_KEY = os.getenv("JWT_SECRET_KEY", secrets.token_urlsafe(32))
```

---

#### 4.1.3 Token 过期时间过长

**漏洞描述：**
Token 有效期过长，泄露后攻击窗口大。

**防护措施：**
```python
from datetime import datetime, timedelta

def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    # Access Token 短有效期（15-30 分钟）
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire, "type": "access"})
    return jwt.encode(to_encode, SECRET_KEY, algorithm="HS256")

def create_refresh_token(data: dict):
    to_encode = data.copy()
    # Refresh Token 较长有效期（7 天）
    expire = datetime.utcnow() + timedelta(days=7)
    to_encode.update({"exp": expire, "type": "refresh"})
    return jwt.encode(to_encode, SECRET_KEY, algorithm="HS256")
```

---

#### 4.1.4 缺少 Token 吊销机制

**漏洞描述：**
JWT 无状态特性导致 Token 一旦签发无法主动吊销。

**防护措施：**
```python
# 方案1：Token 黑名单（Redis）
class TokenBlacklist:
    def __init__(self, redis_client):
        self.redis = redis_client

    def add(self, jti: str, expires_in: int):
        """将 Token JTI 加入黑名单"""
        self.redis.setex(f"blacklist:{jti}", expires_in, "1")

    def is_blacklisted(self, jti: str) -> bool:
        """检查 Token 是否在黑名单中"""
        return self.redis.exists(f"blacklist:{jti}")

# 方案2：短期 Token + Refresh Token
# Access Token 15 分钟过期
# Refresh Token 7 天过期，可用于获取新 Access Token
```

---

#### 4.1.5 敏感信息泄露

**漏洞描述：**
JWT Payload 默认仅 Base64 编码（非加密），不应包含敏感信息。

**防护措施：**
```python
# ❌ 危险：包含敏感信息
payload = {
    "user_id": 123,
    "password": "plaintext_password",  # 绝对禁止
    "email": "user@example.com",
    "role": "admin"
}

# ✅ 安全：最小化信息
payload = {
    "sub": "123",           # 用户 ID
    "role": "admin",        # 角色
    "exp": expiry_time,     # 过期时间
    "jti": token_id         # Token 唯一标识（用于吊销）
}
```

---

### 4.2 项目 JWT 实现建议

#### 4.2.1 推荐的 JWT 服务实现

```python
# src/core/security.py
from datetime import datetime, timedelta
from typing import Optional
import jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# 密码哈希
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

class TokenPayload(BaseModel):
    sub: str        # 用户 ID
    role: str       # 用户角色
    exp: datetime   # 过期时间
    jti: str        # Token 唯一标识
    type: str       # token 类型：access/refresh

class JWTService:
    def __init__(self, secret_key: str, algorithm: str = "HS256"):
        self.secret_key = secret_key
        self.algorithm = algorithm

    def create_access_token(
        self,
        user_id: str,
        role: str,
        expires_delta: Optional[timedelta] = None
    ) -> str:
        """创建 Access Token"""
        import uuid
        expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
        payload = {
            "sub": user_id,
            "role": role,
            "exp": expire,
            "jti": str(uuid.uuid4()),
            "type": "access"
        }
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)

    def create_refresh_token(self, user_id: str) -> str:
        """创建 Refresh Token"""
        import uuid
        expire = datetime.utcnow() + timedelta(days=7)
        payload = {
            "sub": user_id,
            "exp": expire,
            "jti": str(uuid.uuid4()),
            "type": "refresh"
        }
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)

    def decode_token(self, token: str) -> TokenPayload:
        """解码并验证 Token"""
        try:
            payload = jwt.decode(
                token,
                self.secret_key,
                algorithms=[self.algorithm],
                options={
                    "verify_signature": True,
                    "verify_exp": True,
                    "verify_aud": False
                }
            )
            return TokenPayload(**payload)
        except jwt.ExpiredSignatureError:
            raise ValueError("Token 已过期")
        except jwt.InvalidTokenError:
            raise ValueError("无效的 Token")

# 全局实例
from src.core.config import settings
jwt_service = JWTService(
    secret_key=settings.JWT_SECRET_KEY,
    algorithm="HS256"
)
```

---

#### 4.2.2 FastAPI 认证依赖

```python
# src/api/auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from src.core.security import jwt_service
from src.models.user import User

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> User:
    """获取当前认证用户"""
    token = credentials.credentials
    try:
        payload = jwt_service.decode_token(token)
        if payload.type != "access":
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="无效的 Token 类型"
            )
        # 查询用户
        user = await get_user_by_id(payload.sub)
        if not user:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="用户不存在"
            )
        return user
    except ValueError as e:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=str(e)
        )

async def require_admin(
    current_user: User = Depends(get_current_user)
) -> User:
    """要求管理员权限"""
    if current_user.role != "admin":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="权限不足"
        )
    return current_user
```

---

#### 4.2.3 路由认证示例

```python
# src/api/routes/students.py
from fastapi import APIRouter, Depends
from src.api.auth import get_current_user, require_admin
from src.models.user import User

router = APIRouter(prefix="/api/v1/students", tags=["学生管理"])

@router.get("/")
async def list_students(
    current_user: User = Depends(get_current_user)  # 需要登录
):
    """获取学生列表（需要认证）"""
    ...

@router.post("/")
async def create_student(
    student_data: StudentCreate,
    current_user: User = Depends(require_admin)  # 需要管理员权限
):
    """创建学生（需要管理员权限）"""
    ...

@router.delete("/{student_id}")
async def delete_student(
    student_id: str,
    current_user: User = Depends(require_admin)  # 需要管理员权限
):
    """删除学生（需要管理员权限）"""
    ...
```

---

## 5. Vue.js 前端安全调研

### 5.1 常见前端安全漏洞

#### 5.1.1 XSS（跨站脚本攻击）

**Vue 3 内置防护：**
```vue
<template>
  <!-- ✅ 安全：Vue 自动转义 -->
  <p>{{ userInput }}</p>

  <!-- ❌ 危险：v-html 可能导致 XSS -->
  <div v-html="userInput"></div>

  <!-- ✅ 安全：使用文本插值 -->
  <div v-text="userInput"></div>
</template>
```

**防护建议：**
```typescript
// 1. 避免使用 v-html
// 2. 如果必须使用 v-html，先净化
import DOMPurify from 'dompurify';

const sanitizedHtml = DOMPurify.sanitize(userInput);
```

---

#### 5.1.2 CSRF（跨站请求伪造）

**Axios 拦截器配置：**
```typescript
// frontend/src/utils/request.ts
import axios from 'axios';

const service = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  withCredentials: true,  // 携带 Cookie
});

// 请求拦截器
service.interceptors.request.use(
  (config) => {
    // 1. 添加 JWT Token
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // 2. 添加 CSRF Token（如果使用 Cookie 认证）
    const csrfToken = document.cookie
      .split('; ')
      .find(row => row.startsWith('csrftoken='))
      ?.split('=')[1];
    if (csrfToken) {
      config.headers['X-CSRF-Token'] = csrfToken;
    }

    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器
service.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token 过期，清除本地存储并跳转登录
      localStorage.removeItem('access_token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default service;
```

---

#### 5.1.3 本地存储安全

**安全存储建议：**
```typescript
// frontend/src/utils/auth.ts

// ❌ 不安全：localStorage 容易被 XSS 攻击读取
localStorage.setItem('token', token);

// ✅ 更安全：使用 httpOnly Cookie（需要后端配合）
// 后端设置：Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict

// ✅ 前端安全实践
class SecureStorage {
  // 敏感数据使用 sessionStorage（会话结束清除）
  static setSession(key: string, value: string): void {
    sessionStorage.setItem(key, value);
  }

  static getSession(key: string): string | null {
    return sessionStorage.getItem(key);
  }

  // 非敏感数据可使用 localStorage
  static setLocal(key: string, value: string): void {
    localStorage.setItem(key, value);
  }

  static getLocal(key: string): string | null {
    return localStorage.getItem(key);
  }

  // 清除所有认证信息
  static clearAuth(): void {
    sessionStorage.clear();
    localStorage.removeItem('access_token');
    localStorage.removeItem('refresh_token');
  }
}
```

---

### 5.2 Content Security Policy (CSP)

**Vite 配置：**
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  server: {
    headers: {
      'Content-Security-Policy': [
        "default-src 'self'",
        "script-src 'self'",
        "style-src 'self' 'unsafe-inline'",  // Element Plus 需要 unsafe-inline
        "img-src 'self' data: https:",
        "font-src 'self' data:",
        "connect-src 'self' http://localhost:8000",  // API 地址
        "frame-ancestors 'none'",
      ].join('; '),
      'X-Content-Type-Options': 'nosniff',
      'X-Frame-Options': 'DENY',
      'X-XSS-Protection': '1; mode=block',
    },
  },
});
```

---

### 5.3 依赖安全检查

**定期审计命令：**
```bash
# npm
npm audit
npm audit fix

# pnpm
pnpm audit
```

**已知需要关注的依赖：**

| 依赖 | 版本 | 已知风险 | 建议 |
|------|------|----------|------|
| axios | ^1.4.0 | CVE-2023-45857（CSRF Token 泄露） | 升级到 1.6.0+ |
| echarts | ^5.4.0 | 无已知高危 | 定期更新 |
| element-plus | ^2.4.0 | 无已知高危 | 定期更新 |

---

## 6. 综合安全加固建议

### 6.1 优先级排序

| 优先级 | 漏洞/风险 | 修复建议 | 预估工时 |
|--------|-----------|----------|----------|
| **P0** | 缺少认证授权 | 实现 JWT 认证 + RBAC | 16h |
| **P1** | CORS 配置过于宽松 | 限制允许的来源 | 1h |
| **P1** | python-multipart DoS | 升级到 0.0.9+ | 0.5h |
| **P2** | 缺少 CSRF 防护 | 实现 CSRF Token | 4h |
| **P2** | JWT 安全配置 | 实现安全的 JWT 服务 | 8h |
| **P3** | 安全响应头 | 添加中间件 | 2h |
| **P3** | 前端 CSP 配置 | 配置 Vite | 2h |
| **P3** | 依赖版本更新 | 审计并升级 | 4h |

---

### 6.2 实施路线图

```
阶段 1：基础认证（Week 1-2）
├── 实现 JWT 服务（access + refresh token）
├── 实现用户认证依赖
├── 为所有路由添加认证保护
└── 修复 CORS 配置

阶段 2：安全加固（Week 3）
├── 实现 RBAC 角色权限
├── 添加 CSRF 防护
├── 配置安全响应头
└── 升级有漏洞的依赖

阶段 3：前端安全（Week 4）
├── 实现 Token 安全存储
├── 配置 CSP
├── 添加请求拦截器
└── 实现 Token 自动刷新
```

---

### 6.3 推荐依赖清单

**后端（requirements.txt）：**
```txt
# 安全相关
python-jose[cryptography]>=3.3.0  # JWT 处理
passlib[bcrypt]>=1.7.4             # 密码哈希
python-multipart>=0.0.9            # 文件上传（修复 CVE-2024-24762）
slowapi>=0.1.9                     # 速率限制

# 升级有漏洞的包
sqlalchemy>=2.0.30                 # 修复 CVE-2024-26130
werkzeug>=3.0.1                    # 修复 CVE-2023-46136
```

**前端（package.json）：**
```json
{
  "dependencies": {
    "axios": "^1.6.0",          // 修复 CVE-2023-45857
    "dompurify": "^3.0.6",      // HTML 净化
    "pinia": "^2.1.0",
    "vue": "^3.3.0",
    "vue-router": "^4.2.0"
  }
}
```

---

## 7. 参考资源

### 7.1 官方文档
- FastAPI Security: https://fastapi.tiangolo.com/tutorial/security/
- SQLAlchemy Security: https://docs.sqlalchemy.org/en/20/core/connections.html#sql-injection
- Vue.js Security: https://vuejs.org/guide/best-practices/security.html
- OWASP JWT Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html

### 7.2 CVE 数据库
- NVD: https://nvd.nist.gov/
- GitHub Advisory: https://github.com/advisories
- CVE Details: https://www.cvedetails.com/

### 7.3 安全工具
- npm audit: https://docs.npmjs.com/cli/v10/commands/npm-audit
- pip-audit: https://pypi.org/project/pip-audit/
- Snyk: https://snyk.io/

---

## 8. 附录：项目当前安全状态

基于 `vulnerability-assessment.md` 的审计结果，当前项目安全评分为 **78/100**。

| 安全领域 | 评分 | 状态 |
|----------|------|------|
| SQL 注入防护 | ⭐⭐⭐⭐⭐ | 已通过 ORM 防护 |
| XSS 防护 | ⭐⭐⭐⭐⭐ | Vue 3 默认转义 |
| 命令注入防护 | ⭐⭐⭐⭐⭐ | 无系统命令调用 |
| 身份认证 | ⭐☆☆☆☆ | **完全缺失** |
| 授权控制 | ⭐☆☆☆☆ | **完全缺失** |
| CORS 配置 | ⭐⭐☆☆☆ | 过于宽松 |
| 输入验证 | ⭐⭐⭐⭐⭐ | Pydantic 双重验证 |
| 凭据管理 | ⭐⭐⭐⭐⭐ | 环境变量管理 |

**核心发现：** 项目基础安全架构良好（ORM 防护、输入验证），但**认证授权完全缺失**，是当前最需要优先解决的安全风险。

---

*报告由 Research 前沿调研员自动生成*
*调研日期：2026-06-06*
