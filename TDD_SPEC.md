# TDD需求规格说明书 - 启发式AI辅导师

> 版本: 1.1  
> 适用: AI辅助开发 + TDD流程  
> 项目: heuristic-ai-tutor
> 更新: 2025-04-09 - 优化单元测试数量要求

---

## 1. 需求元信息

```yaml
需求ID: REQ-HAIT-001
需求名称: 启发式AI辅导师核心功能
优先级: P0
迭代: Sprint 1-3
负责人: 开发团队
状态: 开发中
```

### 1.1 背景与目标
- **业务背景**: 面向K12学生的苏格拉底式AI讲题助手，通过启发式引导帮助学生自主发现解题思路，而非直接给出答案
- **用户价值**: 
  - 学生：获得个性化引导，真正理解解题思路
  - 家长：了解孩子学习情况，查看学习报告
- **成功指标**: 
  - OCR识别准确率 ≥ 95%（印刷体）/ ≥ 85%（手写体）
  - 错因诊断置信度 ≥ 70%
  - 用户满意度 ≥ 4.0/5.0

### 1.2 范围边界
- **包含**: 用户管理、题目识别、错因诊断、引导对话、学习档案
- **不包含**: 直播授课、作业批改、班级管理

---

## 2. 用户故事

### US-1: 学生注册与登录
```gherkin
作为 学生用户
我希望 通过手机号验证码快速注册登录
以便 使用AI辅导功能
```

**验收标准 (AC):**
```gherkin
AC1: 手机号注册
  Given 用户输入有效手机号（11位）
  When 请求发送验证码
  Then 验证码发送成功，60秒内不可重复发送

AC2: 验证码登录
  Given 用户输入正确验证码
  When 提交登录
  Then 登录成功，返回JWT Token

AC3: 信息完善
  Given 新注册用户
  When 完善年级（1-12）和学科偏好
  Then 信息保存成功，进入首页
```

### US-2: 拍照上传题目
```gherkin
作为 学生用户
我希望 拍照上传题目图片
以便 获得AI辅导
```

**验收标准 (AC):**
```gherkin
AC1: 图片上传
  Given 用户选择或拍摄图片（jpg/png）
  When 图片大小 ≤ 10MB
  Then 上传成功，自动压缩至1MB以内

AC2: 图片预处理
  Given 上传的图片
  When 进行OCR预处理（去噪、二值化、倾斜校正）
  Then 图片质量提升，适合识别

AC3: 大图片处理
  Given 图片大小 > 10MB
  When 尝试上传
  Then 返回错误提示"图片过大，请选择小于10MB的图片"
```

### US-3: 错因诊断
```gherkin
作为 学生用户
我希望 系统分析我的错误答案并诊断错因
以便 了解自己的薄弱环节
```

**验收标准 (AC):**
```gherkin
AC1: 知识点匹配
  Given 已识别的题目文本
  When 执行知识点匹配
  Then 返回Top-3相关知识点

AC2: 错因诊断-规则引擎
  Given 题目、学生答案和知识点
  When 规则引擎匹配成功且置信度 ≥ 0.7
  Then 返回Top-3错因及置信度

AC3: 错因诊断-BERT模型
  Given 规则引擎未匹配成功
  When BERT模型分类且置信度 ≥ 0.7
  Then 返回Top-3错因及置信度

AC4: 错因诊断-LLM兜底
  Given 前两种策略置信度均 < 0.7
  When 调用大模型诊断
  Then 返回LLM诊断结果

AC5: 历史错因加权
  Given 该用户近30天有错因记录
  When 执行诊断时
  Then 相同错因加权+0.1
```

### US-4: 引导对话
```gherkin
作为 学生用户
我希望 通过苏格拉底式对话获得引导
以便 自己发现解题思路
```

**验收标准 (AC):**
```gherkin
AC1: 开始引导会话
  Given 已完成错因诊断
  When 请求开始引导
  Then 创建会话，返回初始引导问题

AC2: 会话状态保持
  Given 正在进行中的会话
  When 用户发送回复
  Then 会话状态更新，返回下一步引导

AC3: 意图识别-选项点击
  Given 用户点击预设选项
  When 匹配选项ID
  Then 直接进入对应分支

AC4: 意图识别-自由文本
  Given 用户输入自由文本
  When 执行意图识别（小模型+LLM兜底）
  Then 识别用户意图，进入对应分支

AC5: 会话超时
  Given 会话超过30分钟无活动
  When 再次访问该会话
  Then 会话已过期，提示重新开始

AC6: 禁止直接给答案
  Given 任何对话节点
  When 生成话术时
  Then 必须遵循苏格拉底式引导，不直接给出答案
```

### US-5: 家长绑定与报告
```gherkin
作为 家长用户
我希望 绑定孩子的账号并查看学习报告
以便 了解孩子的学习情况
```

**验收标准 (AC):**
```gherkin
AC1: 生成绑定码
  Given 学生账号
  When 请求生成绑定码
  Then 生成6位数字绑定码，有效期30分钟

AC2: 家长绑定
  Given 家长输入有效绑定码
  When 提交绑定请求
  Then 绑定成功，建立家长-学生关系

AC3: 查看学习报告
  Given 已绑定学生的家长
  When 请求查看学习报告（周/月）
  Then 返回学习统计数据和思维成长曲线
```

### US-6: 学习记录与反馈
```gherkin
作为 学生用户
我希望 查看学习记录并评价引导效果
以便 追踪自己的学习进步
```

**验收标准 (AC):**
```gherkin
AC1: 记录学习过程
  Given 完成一次辅导会话
  When 会话结束时
  Then 保存完整学习记录（题目、诊断、对话过程）

AC2: 用户评价
  Given 会话结束
  When 用户提交1-5分评分和可选反馈
  Then 评价记录保存成功

AC3: 低置信度触发审核
  Given 诊断置信度 < 0.7 或 用户评分 ≤ 2
  When 会话结束后
  Then 自动生成人工审核任务
```

---

## 3. 功能规格

### 3.1 用例清单

| 用例ID | 用例名称 | 优先级 | 对应AC | 测试类型 |
|--------|----------|--------|--------|----------|
| UC-01 | 用户注册 | P0 | US-1 AC1 | 单元测试 |
| UC-02 | 验证码登录 | P0 | US-1 AC2 | 单元测试 |
| UC-03 | 图片上传与压缩 | P0 | US-2 AC1 | 单元测试+集成测试 |
| UC-04 | 图片预处理 | P0 | US-2 AC2 | 单元测试 |
| UC-05 | 知识点匹配 | P0 | US-3 AC1 | 单元测试 |
| UC-06 | 规则引擎诊断 | P0 | US-3 AC2 | 单元测试 |
| UC-07 | BERT模型诊断 | P0 | US-3 AC3 | 集成测试 |
| UC-08 | LLM兜底诊断 | P1 | US-3 AC4 | 集成测试 |
| UC-09 | 历史错因加权 | P1 | US-3 AC5 | 单元测试 |
| UC-10 | 开始引导会话 | P0 | US-4 AC1 | 单元测试 |
| UC-11 | 会话状态管理 | P0 | US-4 AC2 | 单元测试 |
| UC-12 | 意图识别 | P0 | US-4 AC3,AC4 | 单元测试+集成测试 |
| UC-13 | 话术生成 | P0 | US-4 AC6 | 单元测试+集成测试 |
| UC-14 | 会话超时处理 | P1 | US-4 AC5 | 单元测试 |
| UC-15 | 家长绑定 | P1 | US-5 AC1,AC2 | 单元测试 |
| UC-16 | 学习报告生成 | P1 | US-5 AC3 | 单元测试 |
| UC-17 | 学习记录保存 | P0 | US-6 AC1 | 单元测试 |
| UC-18 | 用户评价提交 | P0 | US-6 AC2 | 单元测试 |
| UC-19 | 人工审核任务创建 | P1 | US-6 AC3 | 单元测试 |

### 3.2 详细用例

#### UC-01: 用户注册

```gherkin
@priority:high @type:unit
功能: 用户通过手机号注册

  场景: 有效手机号注册
    Given 用户输入手机号 "13800138000"
    And 该手机号未注册
    When 请求发送验证码
    Then 返回成功状态
    And 数据库创建验证码记录
    And 60秒内不可重复发送

  场景: 已注册手机号
    Given 用户输入手机号 "13800138000"
    And 该手机号已注册
    When 请求发送验证码
    Then 返回错误码 "PHONE_ALREADY_REGISTERED"
    And 提示"该手机号已注册，请直接登录"

  场景: 无效手机号格式
    Given 用户输入手机号 "12345"
    When 请求发送验证码
    Then 返回错误码 "INVALID_PHONE_FORMAT"
    And 提示"请输入正确的11位手机号"

  场景大纲: 验证码验证
    Given 用户输入手机号 "13800138000"
    And 验证码为 "123456"
    When 提交验证码 "<input_code>"
    Then 期望 "<result>"
    
    例子:
      | input_code | result  | 说明 |
      | 123456     | success | 正确验证码 |
      | 654321     | error   | 错误验证码 |
      |            | error   | 空验证码 |
```

#### UC-05: 知识点匹配

```gherkin
@priority:high @type:unit
功能: 根据题目文本匹配知识点

  场景: 关键词匹配成功
    Given 题目文本 "解一元二次方程 x² + 2x + 1 = 0"
    And 知识点库包含 "一元二次方程"
    When 执行知识点匹配
    Then 返回知识点列表
      | knowledgePointId | name | confidence |
      | KP001            | 一元二次方程 | 0.95 |
      | KP002            | 因式分解 | 0.75 |
      | KP003            | 完全平方公式 | 0.60 |

  场景: 向量相似度匹配
    Given 题目文本 "求函数 f(x) = x² 在 x=1 处的导数"
    And 关键词匹配失败
    When 执行向量相似度检索
    Then 返回相似度Top-3知识点
      | knowledgePointId | name | similarity |
      | KP010            | 导数定义 | 0.92 |
      | KP011            | 函数极限 | 0.75 |
      | KP012            | 幂函数求导 | 0.70 |

  场景: 无匹配知识点
    Given 题目文本 "这是一个无法识别的题目"
    When 执行知识点匹配
    Then 返回空列表
    And 触发人工标注流程
```

#### UC-06: 规则引擎诊断

```gherkin
@priority:high @type:unit
功能: 基于规则匹配诊断错因

  场景: 规则匹配成功
    Given 题目 "3x + 5 = 20"
    And 学生答案 "x = 5"
    And 知识点 "一元一次方程"
    And 规则库包含规则 "常数项移项未变号"
    When 执行规则引擎匹配
    Then 返回诊断结果
      | misconceptionId | name | confidence |
      | MC001           | 移项变号规则错误 | 0.85 |

  场景: 置信度低于阈值
    Given 题目 "2x + 3 = 7"
    And 学生答案 "x = 3"
    When 执行规则引擎匹配
    Then 最高置信度 < 0.7
    And 进入BERT模型诊断流程
```

#### UC-10: 开始引导会话

```gherkin
@priority:high @type:unit
功能: 创建并初始化引导会话

  场景: 成功开始会话
    Given 用户ID 10001
    And 错因ID "MC001"
    And 题目ID "Q001"
    When 请求开始引导
    Then 创建会话记录
      | sessionId | userId | misconceptionId | currentNodeId |
      | S001      | 10001  | MC001           | START_NODE    |
    And 返回引导响应
      | nodeType | content | options |
      | START    | "让我们一起看看这道题..." | [{id:"O1",text:"好的"}] |
    And Redis存储会话状态，TTL=1800秒

  场景: 错因不存在
    Given 用户ID 10001
    And 错因ID "MC_INVALID"
    When 请求开始引导
    Then 返回错误码 "MISCONCEPTION_NOT_FOUND"
    And 错误级别 ERROR
```

#### UC-12: 意图识别

```gherkin
@priority:high @type:unit
功能: 识别用户回复的意图

  场景: 预设选项匹配
    Given 当前节点选项 [{id:"OPT_1",text:"会了"},{id:"OPT_2",text:"不会"}]
    When 用户选择 "OPT_1"
    Then 返回意图 "UNDERSTOOD"
    And 下一节点为对应分支

  场景: 自由文本意图分类-小模型
    Given 用户输入 "我还是不太明白"
    When 小模型分类
    Then 返回意图 "NEED_HELP"
    And 置信度 ≥ 0.8

  场景: 自由文本意图分类-LLM兜底
    Given 用户输入 "这个公式怎么来的"
    And 小模型置信度 < 0.8
    When 调用LLM识别意图
    Then 返回意图 "ASK_EXPLANATION"
    And 缓存结果30分钟
```

#### UC-17: 学习记录保存

```gherkin
@priority:high @type:unit
功能: 保存完整学习过程记录

  场景: 正常保存学习记录
    Given 会话ID "S001"
    And 用户ID 10001
    And 题目ID "Q001"
    And 错因ID "MC001"
    And 最终结果为 "SOLVED"
    And 对话历史 [{type:"bot",content:"..."},{type:"user",content:"..."}]
    When 结束会话并保存记录
    Then 数据库创建学习记录
      | recordId | userId | questionId | finalResult |
      | R001     | 10001  | Q001       | SOLVED      |
    And 更新用户错因统计表
      | userId | misconceptionId | occurrenceCount |
      | 10001  | MC001           | +1              |

  场景: 数据验证失败
    Given 用户ID为空
    When 尝试保存记录
    Then 抛出 ValidationException
    And 错误码 "PARAM_INVALID"
    And 错误消息 "userId不能为空"
```

---

## 4. 接口规格

### 4.1 用户服务接口

#### 用户注册 - UserController.register
```java
/**
 * 用户注册
 * 
 * @param request 包含手机号、验证码、年级
 * @return 用户信息 + JWT Token
 * @throws BusinessException 手机号已注册/验证码错误
 * 
 * 测试要点:
 *   - TC1: 正常注册
 *   - TC2: 手机号已注册
 *   - TC3: 验证码错误
 *   - TC4: 手机号格式无效
 */
ApiResponse<UserResponse> register(UserRegisterRequest request);
```

```yaml
接口: POST /api/v1/users/register
名称: 用户注册

Request:
  Content-Type: application/json
  Body:
    type: object
    required: [phone, verificationCode, grade]
    properties:
      phone:
        type: string
        pattern: "^1[3-9]\\d{9}$"
        example: "13800138000"
      verificationCode:
        type: string
        length: 6
        example: "123456"
      grade:
        type: integer
        range: [1, 12]
        example: 8
      subjects:
        type: array
        items:
          type: string
          enum: [MATH, PHYSICS, CHEMISTRY]

Response:
  200:
    description: 注册成功
    body:
      code: "SUCCESS"
      data:
        id: 10001
        phone: "13800138000"
        grade: 8
        token: "eyJhbGciOiJIUzI1NiIs..."
  
  400:
    description: 参数错误
    body:
      code: "PARAM_INVALID"
      message: "手机号格式错误"
  
  409:
    description: 冲突
    body:
      code: "PHONE_ALREADY_REGISTERED"
      message: "该手机号已注册"

TestCases:
  - TC-API-001: 正常请求 -> 200 + token
  - TC-API-002: 缺少必填字段 -> 400
  - TC-API-003: 手机号格式错误 -> 400
  - TC-API-004: 验证码错误 -> 400
  - TC-API-005: 手机号已注册 -> 409
```

#### 用户登录 - UserController.login
```yaml
接口: POST /api/v1/users/login
名称: 用户登录

Request:
  Content-Type: application/json
  Body:
    oneOf:
      - $ref: "#/PhoneLogin"
      - $ref: "#/WechatLogin"

PhoneLogin:
  type: object
  required: [phone, verificationCode]
  properties:
    phone:
      type: string
    verificationCode:
      type: string

WechatLogin:
  type: object
  required: [wxCode]
  properties:
    wxCode:
      type: string
      description: 微信授权code

Response:
  200:
    data:
      user: UserResponse
      token: string
      expiresIn: 7200  # 2小时
```

### 4.2 错因诊断服务接口

#### 错因诊断 - DiagnosisController.diagnose
```java
/**
 * 执行错因诊断
 * 
 * @param request 包含题目文本、学生答案、用户ID（可选，用于历史加权）
 * @return Top-3错因列表，按置信度排序
 * @throws BusinessException 参数无效/知识点匹配失败
 * 
 * 测试要点:
 *   - TC1: 规则引擎匹配成功
 *   - TC2: BERT模型匹配成功
 *   - TC3: LLM兜底诊断
 *   - TC4: 历史错因加权生效
 *   - TC5: 所有策略置信度均低
 */
ApiResponse<List<DiagnosisResult>> diagnose(DiagnosisRequest request);
```

```yaml
接口: POST /api/v1/diagnosis
名称: 错因诊断

Request:
  Body:
    type: object
    required: [questionText, studentAnswer]
    properties:
      questionText:
        type: string
        maxLength: 2000
        example: "解方程 3x + 5 = 20"
      studentAnswer:
        type: string
        maxLength: 1000
        example: "x = 5"
      studentId:
        type: integer
        description: 用于历史错因加权

Response:
  200:
    data:
      type: array
      maxItems: 3
      items:
        type: object
        properties:
          misconceptionId:
            type: string
          name:
            type: string
            example: "移项变号规则错误"
          confidence:
            type: number
            range: [0, 1]
          guidanceTreeId:
            type: string
          description:
            type: string

TestCases:
  - TC-API-006: 规则引擎匹配成功 -> 返回结果
  - TC-API-007: 所有策略置信度低 -> 返回LLM结果+低置信度标记
  - TC-API-008: 题目文本为空 -> 400
  - TC-API-009: 文本超长 -> 400
```

### 4.3 引导对话服务接口

#### 开始引导 - GuidanceController.startGuidance
```yaml
接口: POST /api/v1/guidance/start
名称: 开始引导会话

Request:
  Body:
    type: object
    required: [userId, misconceptionId, questionId]
    properties:
      userId:
        type: integer
      misconceptionId:
        type: string
      questionId:
        type: string
      context:
        type: object
        description: 额外上下文

Response:
  200:
    data:
      sessionId: "S001"
      nodeType: "START"
      content: "让我们一起看看这道题..."
      options:
        - id: "OPT_1"
          text: "好的，开始吧"
        - id: "OPT_2"
          text: "我想先看看题目"
      guidanceTreeId: "GT001"

  404:
    description: 错因或题目不存在
```

#### 下一步引导 - GuidanceController.nextStep
```yaml
接口: POST /api/v1/guidance/next
名称: 获取下一步引导

Request:
  Body:
    type: object
    required: [sessionId, userResponse]
    properties:
      sessionId:
        type: string
      userResponse:
        type: object
        properties:
          type:
            type: string
            enum: [OPTION, TEXT]
          optionId:
            type: string
            description: type=OPTION时必填
          text:
            type: string
            description: type=TEXT时必填

Response:
  200:
    data:
      sessionId: "S001"
      nodeType: "QUESTION"  # START, QUESTION, HINT, FEEDBACK, END
      content: "你能告诉我这一步的思路吗？"
      options: [...]  # 可能为空，表示自由输入
      isEnd: false

  410:
    description: 会话已过期
    body:
      code: "SESSION_EXPIRED"
```

#### 结束会话 - GuidanceController.endSession
```yaml
接口: POST /api/v1/guidance/end
名称: 结束引导会话

Request:
  Body:
    type: object
    required: [sessionId, finalResult]
    properties:
      sessionId:
        type: string
      finalResult:
        type: string
        enum: [SOLVED, UNSOLVED, ABANDONED]
      userRating:
        type: integer
        range: [1, 5]
      feedback:
        type: string
        maxLength: 500

Response:
  200:
    description: 会话结束，记录已保存
```

### 4.4 题目识别服务接口

#### 图片识别 - OCRController.recognize
```yaml
接口: POST /api/v1/ocr/recognize
名称: 图片OCR识别

Request:
  Content-Type: multipart/form-data
  Body:
    image:
      type: file
      format: [jpg, jpeg, png]
      maxSize: 10MB

Response:
  200:
    data:
      questionText: "解方程 3x + 5 = 20"
      formulaLatex: "3x + 5 = 20"
      studentAnswer: "x = 5"
      confidence: 0.95
      imageUrl: "https://..."

  400:
    description: 图片格式错误或过大
```

### 4.5 家长服务接口

#### 生成绑定码 - UserController.generateBindCode
```yaml
接口: POST /api/v1/users/bind-code
名称: 生成家长绑定码

Request:
  Headers:
    Authorization: Bearer {token}

Response:
  200:
    data:
      bindCode: "123456"
      expiresAt: "2024-01-01T12:30:00Z"
      ttl: 1800  # 30分钟
```

#### 家长绑定 - UserController.bindParent
```yaml
接口: POST /api/v1/users/bind-parent
名称: 家长绑定学生

Request:
  Body:
    bindCode: "123456"

Response:
  200:
    description: 绑定成功
  400:
    description: 绑定码无效或过期
```

---

## 5. 数据规格

### 5.1 核心数据模型

#### 用户表 (users)
```yaml
实体: User
表名: users

字段:
  - name: id
    type: BIGINT
    nullable: false
    primaryKey: true
    autoIncrement: true
    
  - name: phone
    type: VARCHAR(11)
    nullable: true
    unique: true
    validation: "^1[3-9]\\d{9}$"
    
  - name: wx_openid
    type: VARCHAR(64)
    nullable: true
    unique: true
    
  - name: nickname
    type: VARCHAR(50)
    nullable: true
    
  - name: avatar_url
    type: VARCHAR(500)
    nullable: true
    
  - name: grade
    type: INTEGER
    nullable: true
    range: [1, 12]
    
  - name: subjects
    type: JSON
    nullable: true
    example: '["MATH", "PHYSICS"]'
    
  - name: parent_id
    type: BIGINT
    nullable: true
    foreignKey: users.id
    
  - name: created_at
    type: TIMESTAMP
    nullable: false
    default: CURRENT_TIMESTAMP
    
  - name: updated_at
    type: TIMESTAMP
    nullable: false
    default: CURRENT_TIMESTAMP

索引:
  - name: idx_phone
    fields: [phone]
    type: unique
  - name: idx_wx_openid
    fields: [wx_openid]
    type: unique
```

#### 学习记录表 (learning_records)
```yaml
实体: LearningRecord
表名: learning_records

字段:
  - name: record_id
    type: VARCHAR(32)
    primaryKey: true
    
  - name: user_id
    type: BIGINT
    nullable: false
    foreignKey: users.id
    
  - name: question_id
    type: VARCHAR(32)
    nullable: false
    
  - name: question_text
    type: TEXT
    nullable: false
    
  - name: student_answer
    type: TEXT
    nullable: true
    
  - name: final_result
    type: ENUM('SOLVED', 'UNSOLVED', 'ABANDONED')
    nullable: false
    
  - name: diagnosis_misconception_id
    type: VARCHAR(32)
    nullable: true
    
  - name: diagnosis_confidence
    type: DECIMAL(3,2)
    nullable: true
    range: [0, 1]
    
  - name: guidance_tree_id
    type: VARCHAR(32)
    nullable: true
    
  - name: interaction_log
    type: JSON
    nullable: true
    description: 对话历史记录
    
  - name: user_rating
    type: INTEGER
    nullable: true
    range: [1, 5]
    
  - name: created_at
    type: TIMESTAMP
    nullable: false
    default: CURRENT_TIMESTAMP

索引:
  - name: idx_user_created
    fields: [user_id, created_at]
    type: normal
  - name: idx_question
    fields: [question_id]
    type: normal
```

#### 错因表 (misconceptions)
```yaml
实体: Misconception
表名: misconceptions

字段:
  - name: misconception_id
    type: VARCHAR(32)
    primaryKey: true
    
  - name: name
    type: VARCHAR(100)
    nullable: false
    
  - name: description
    type: TEXT
    nullable: true
    
  - name: category
    type: ENUM('CONCEPT', 'CALCULATION', 'LOGIC', 'CARELESS')
    nullable: false
    
  - name: subject
    type: VARCHAR(20)
    nullable: false
    
  - name: difficulty_level
    type: INTEGER
    nullable: true
    range: [1, 5]
    
  - name: knowledge_point_ids
    type: JSON
    nullable: true
    description: 关联知识点ID列表
    
  - name: diagnosis_rule
    type: JSON
    nullable: true
    description: 规则引擎配置
    
  - name: guidance_tree_id
    type: VARCHAR(32)
    nullable: false
    
  - name: confidence_threshold
    type: DECIMAL(3,2)
    nullable: false
    default: 0.70
```

#### 用户错因统计表 (user_misconception_stats)
```yaml
实体: UserMisconceptionStat
表名: user_misconception_stats

字段:
  - name: id
    type: BIGINT
    primaryKey: true
    autoIncrement: true
    
  - name: user_id
    type: BIGINT
    nullable: false
    foreignKey: users.id
    
  - name: misconception_id
    type: VARCHAR(32)
    nullable: false
    
  - name: occurrence_count
    type: INTEGER
    nullable: false
    default: 0
    
  - name: first_occurrence
    type: TIMESTAMP
    nullable: true
    
  - name: last_occurrence
    type: TIMESTAMP
    nullable: true
    
  - name: solved_count
    type: INTEGER
    nullable: false
    default: 0

索引:
  - name: idx_user_misconception
    fields: [user_id, misconception_id]
    type: unique
```

### 5.2 Redis数据模型

#### 会话状态 (dialogue_session)
```yaml
Key: "dialogue:session:{sessionId}"
Type: Hash
TTL: 1800  # 30分钟

Fields:
  sessionId: "S001"
  userId: "10001"
  misconceptionId: "MC001"
  currentNodeId: "NODE_1"
  history: "[...]"  # JSON序列化
  createdAt: "2024-01-01T10:00:00Z"
  lastActivity: "2024-01-01T10:05:00Z"
```

#### 验证码 (verification_code)
```yaml
Key: "verify:code:{phone}"
Type: String
TTL: 300  # 5分钟

Value: "123456"
```

#### 绑定码 (bind_code)
```yaml
Key: "bind:code:{code}"
Type: Hash
TTL: 1800  # 30分钟

Fields:
  studentId: "10001"
  expiresAt: "2024-01-01T12:30:00Z"
```

### 5.3 测试数据集

```yaml
数据集: DS-USER-001
描述: 标准用户测试数据

records:
  - id: 10001
    phone: "13800138001"
    nickname: "测试学生1"
    grade: 8
    subjects: ["MATH", "PHYSICS"]
    created_at: "2024-01-01T00:00:00Z"
  
  - id: 10002
    phone: "13800138002"
    nickname: "测试学生2"
    grade: 9
    subjects: ["MATH"]
    created_at: "2024-01-02T00:00:00Z"

边界数据:
  - name: "最小年级"
    value: 1
  - name: "最大年级"
    value: 12
  - name: "边界手机号"
    value: "19999999999"
```

```yaml
数据集: DS-MISCONCEPTION-001
描述: 错因测试数据

records:
  - misconception_id: "MC001"
    name: "移项变号规则错误"
    category: "CONCEPT"
    subject: "MATH"
    confidence_threshold: 0.70
  
  - misconception_id: "MC002"
    name: "一元二次方程求根公式记错"
    category: "CONCEPT"
    subject: "MATH"
    confidence_threshold: 0.70
```

---

## 6. 错误处理规格

### 6.1 错误码定义

| 错误码 | 级别 | 描述 | 用户提示 | 日志内容 |
|--------|------|------|----------|----------|
| SUCCESS | INFO | 操作成功 | - | - |
| PARAM_INVALID | WARN | 参数验证失败 | 输入参数有误，请检查 | 参数字段和实际值 |
| PHONE_ALREADY_REGISTERED | WARN | 手机号已注册 | 该手机号已注册，请直接登录 | 手机号 |
| INVALID_PHONE_FORMAT | WARN | 手机号格式错误 | 请输入正确的11位手机号 | 输入值 |
| VERIFICATION_CODE_ERROR | WARN | 验证码错误 | 验证码错误，请重新输入 | 输入值vs期望值 |
| VERIFICATION_CODE_EXPIRED | WARN | 验证码过期 | 验证码已过期，请重新获取 | - |
| USER_NOT_FOUND | WARN | 用户不存在 | 用户不存在 | userId |
| SESSION_EXPIRED | WARN | 会话已过期 | 会话已过期，请重新开始 | sessionId |
| MISCONCEPTION_NOT_FOUND | ERROR | 错因不存在 | 系统错误，请稍后重试 | misconceptionId |
| KNOWLEDGE_POINT_NOT_FOUND | WARN | 知识点未匹配 | 暂时无法识别该题目类型 | - |
| OCR_FAILED | ERROR | OCR识别失败 | 图片识别失败，请重新上传 | 异常堆栈 |
| LLM_SERVICE_ERROR | ERROR | LLM服务异常 | 服务繁忙，请稍后重试 | 异常堆栈 |
| INTERNAL_ERROR | ERROR | 内部错误 | 系统繁忙，请稍后重试 | 异常堆栈 |

### 6.2 异常场景

```gherkin
场景: 数据库连接超时
  When 数据库连接超时
  Then 返回错误码 "INTERNAL_ERROR"
  And 用户提示 "服务繁忙，请稍后重试"
  And 记录 ERROR 日志，包含连接池状态
  And 触发告警（连续3次）

场景: Redis连接失败
  When Redis连接失败
  Then 降级使用内存缓存（会话保持时间缩短至5分钟）
  And 记录 WARN 日志
  And 触发告警

场景: LLM服务超时
  When LLM调用超过30秒
  Then 返回超时错误
  And 使用默认话术模板
  And 记录 ERROR 日志
  And 触发告警

场景: 重复提交
  Given 同一请求在5秒内重复提交（通过Redis幂等键判断）
  When 接收到重复请求
  Then 返回上一次的响应结果
  And 记录 INFO 日志
```

---

## 7. 非功能需求

### 7.1 性能要求

```yaml
指标:
  - name: 用户注册接口
    target: "P95 < 500ms"
    testMethod: "并发100用户"
  
  - name: 用户登录接口
    target: "P95 < 300ms"
    testMethod: "并发100用户"
  
  - name: 图片上传接口
    target: "P95 < 3s"
    testMethod: "并发20用户"
  
  - name: OCR识别接口
    target: "P95 < 3s"
    testMethod: "并发20用户"
  
  - name: 错因诊断接口
    target: "P95 < 2s"
    testMethod: "并发50用户"
  
  - name: 引导生成接口
    target: "P95 < 1s"
    testMethod: "并发50用户"
  
  - name: 数据库查询
    target: "单表查询 < 50ms"
    testMethod: "EXPLAIN ANALYZE"
  
  - name: 系统可用性
    target: "≥ 99.9%"
    
  - name: 并发用户数
    target: "支持10万DAU"
```

### 7.2 安全要求

- [x] 输入参数校验（防SQL注入、XSS）
- [x] 敏感数据加密存储（密码AES-256，数据库字段级加密）
- [x] API鉴权验证（JWT Token）
- [x] 敏感操作审计日志（登录、密码修改、绑定操作）
- [x] 传输层加密（TLS 1.3）
- [x] 未成年人隐私保护（符合COPPA）
- [x] 内容安全审核（输出话术过滤）
- [x] 防暴力破解（验证码错误5次锁定15分钟）
- [x] API限流（每分钟100次/用户）

### 7.3 可测试性要求

- [x] 核心逻辑单元测试覆盖率 > 80%
- [x] 所有public方法都有对应测试
- [x] 集成测试覆盖主流程（注册→登录→诊断→引导）
- [x] 边界值测试全覆盖（手机号格式、年级范围、评分范围）
- [x] 异常分支测试（数据库超时、Redis故障、LLM超时）
- [x] Mock外部依赖（OCR、LLM、微信API）

---

## 8. 测试策略

### 8.1 测试金字塔

```
        /\
       /  \  E2E测试 (2个主流程)
      /____\      - 完整辅导流程
     /      \     - 家长查看报告
    /________\
   /          \  集成测试 (API + DB + Redis)
  /            \    - 用户模块集成
 /______________\   - 诊断模块集成
/                \  - 引导模块集成
   单元测试 (核心业务逻辑)
      - Service层: User/Diagnosis/Guidance
      - Repository层: CRUD操作
      - 工具类: JWT/Utils
```

### 8.2 测试清单

| 测试ID | 类型 | 描述 | 自动化 | 优先级 |
|--------|------|------|--------|--------|
| UT-001 | 单元 | UserService.register - 正常注册 | 是 | P0 |
| UT-002 | 单元 | UserService.register - 手机号已注册 | 是 | P0 |
| UT-003 | 单元 | UserService.login - 验证码登录 | 是 | P0 |
| UT-004 | 单元 | UserRepository - CRUD操作 | 是 | P0 |
| UT-005 | 单元 | DiagnosisService.diagnose - 规则引擎 | 是 | P0 |
| UT-006 | 单元 | DiagnosisService.diagnose - BERT模型 | 是 | P0 |
| UT-007 | 单元 | DiagnosisService - 历史错因加权 | 是 | P1 |
| UT-008 | 单元 | GuidanceService.startGuidance | 是 | P0 |
| UT-009 | 单元 | GuidanceService.nextStep - 选项匹配 | 是 | P0 |
| UT-010 | 单元 | GuidanceService.nextStep - 意图识别 | 是 | P0 |
| UT-011 | 单元 | GuidanceService - 会话超时处理 | 是 | P1 |
| UT-012 | 单元 | JwtUtil - Token生成与验证 | 是 | P0 |
| UT-013 | 单元 | OCRService - 图片压缩 | 是 | P0 |
| IT-001 | 集成 | 用户模块API端对端 | 是 | P0 |
| IT-002 | 集成 | 诊断模块API端对端 | 是 | P0 |
| IT-003 | 集成 | 引导模块API端对端 | 是 | P0 |
| IT-004 | 集成 | 数据库事务一致性 | 是 | P0 |
| IT-005 | 集成 | Redis缓存一致性 | 是 | P1 |
| E2E-01 | E2E | 完整辅导流程 | 是 | P1 |
| E2E-02 | E2E | 家长绑定与报告查看 | 是 | P2 |

### 8.3 Mock策略

```yaml
需要Mock的外部依赖:
  - OCR服务:
      工具: Mockito
      行为: 返回预设RecognitionResult
  
  - BERT模型服务:
      工具: Mockito
      行为: 根据输入返回对应错因和置信度
  
  - LLM服务:
      工具: Mockito
      行为: 返回预设话术内容
  
  - 微信API:
      工具: WireMock
      行为: 模拟code换取openid流程
  
  - 短信服务:
      工具: Mockito
      行为: 记录调用参数，模拟发送成功

测试数据库:
  - 类型: H2内存数据库
  - 用途: Repository层测试
  - 初始化: Flyway迁移脚本

测试缓存:
  - 类型: Embedded Redis
  - 用途: 会话状态测试
  - 配置: 独立端口，测试后清理
```

### 8.4 单元测试最低要求

每个被测方法必须包含：
- [ ] 至少1个正例测试（Happy Path）
- [ ] 至少1个反例测试（Invalid Input / Error Case）
- [ ] 所有边界值的独立测试（最小值、最大值、空值、null、超长/超短）
- [ ] 异常分支的独立测试

**最低测试数计算公式：**
```
最低测试数 = (参数数量 × 3) + (分支数量 × 2) + (异常路径数量 × 1)
```

**示例：**
| 方法签名 | 参数 | 边界/异常 | 最低测试数 |
|----------|------|-----------|------------|
| `validatePhone(String phone)` | 1个 | null, 空串, 11位边界, 格式错误 | 5-6个 |
| `register(UserDTO dto)` | 1个复合参数 | 各字段边界组合 | 8-10个 |
| `diagnose(DiagnosisRequest req)` | 1个复合参数 | null, 空文本, 超长文本, 知识点不匹配 | 6-8个 |
| `startGuidance(Long userId, String misconceptionId)` | 2个 | null参数, 空串, 无效ID | 6-8个 |

---

## 9. 依赖与前置条件

### 9.1 外部依赖
- [x] PostgreSQL 14+（已完成）
- [x] Redis 6+（已完成）
- [ ] Neo4j（知识图谱模块，Phase 2）
- [ ] Milvus（向量数据库，Phase 2）
- [x] PaddleOCR/Mathpix（OCR服务，已集成）
- [x] Moonshot/OpenAI API（LLM服务，已集成）

### 9.2 内部依赖
- [x] 用户模块（已完成）
- [x] 题目识别模块（已完成）
- [x] 错因诊断模块（已完成）
- [x] 引导对话模块（已完成）
- [ ] 知识图谱模块（Phase 2）
- [ ] 学习报告模块（Phase 2）

### 9.3 数据依赖
- [x] 知识点库初始化数据
- [x] 错因库初始化数据
- [x] 引导决策树配置

---

## 10. 附录

### 10.1 变更历史

| 版本 | 日期 | 作者 | 变更内容 |
|------|------|------|----------|
| 1.1 | 2025-04-09 | 开发团队 | 优化单元测试要求：明确每个public方法至少3-5个测试，增加正例/反例/边界测试要求，补充测试数量计算公式 |
| 1.0 | 2025-04-08 | 开发团队 | 初始版本，基于PRD整理 |

### 10.2 参考文档
- [产品需求文档](docs/02-Product-Requirements-Document.md)
- [模块详细设计](docs/04-Module-Detailed-Design.md)
- [QA协作流程规范](docs/QA_COLLABORATION_WORKFLOW.md)

### 10.3 术语表

| 术语 | 定义 |
|------|------|
| 苏格拉底式引导 | 通过提问而非直接给答案的方式，引导学生自主发现解题思路 |
| 错因 | 学生解题错误的根本原因，如概念理解错误、计算粗心等 |
| 引导决策树 | 根据错因定义的多轮对话流程结构 |
| 意图识别 | 理解用户自由文本回复的意图，如"理解了"、"需要帮助"等 |
| 知识点 | 学科知识的最小单元，如"一元二次方程"、"导数定义"等 |

---

## AI开发提示词模板

```markdown
基于以下需求规格，请为启发式AI辅导师项目生成代码：

1. **生成单元测试代码**
   **⚠️ 数量要求：每个public方法至少3-5个测试方法**
   - 正例测试：正常输入，期望正确输出
   - 反例测试：无效输入，期望抛出异常/错误返回
   - 边界测试：最小值、最大值、空值、null、超长字符串等
   - 并发测试（如适用）：多线程安全验证
   - 安全测试（如适用）：SQL注入、XSS等防护验证
   - 覆盖所有AC场景（US-1到US-6）
   - 包含边界值测试（手机号格式、年级范围、置信度阈值）
   - 使用JUnit5 + Mockito + AssertJ
   - 遵循Given-When-Then结构
   - 使用@ParameterizedTest进行参数化测试
   - **测试命名规范：`should{期望结果}When{条件}`**
   - **示例：`shouldThrowExceptionWhenPhoneIsNull`、 `shouldReturnSuccessWhenValidInput`**
   - **最低测试数 = (参数数量 × 3) + (分支数量 × 2) + (异常路径数量 × 1)**

2. **生成实现代码**
   - 先写骨架（接口/类定义）
   - 实现最小可用版本（MVP）
   - 确保所有测试通过
   - 遵循Spring Boot最佳实践
   - 使用Lombok简化代码
   - 添加必要的日志记录（SLF4J）
   - 标注事务传播行为

3. **生成集成测试**
   - 使用@SpringBootTest
   - 测试API端点（@AutoConfigureMockMvc）
   - 使用H2数据库和Embedded Redis
   - 验证数据持久化和缓存一致性

4. **代码规范**
   - 包结构: com.heuristic.aitutor.{module}
   - 模块: controller/service/repository/entity/dto/exception
   - 统一使用ApiResponse包装响应
   - 异常使用BusinessException
   - 使用@Transactional保证事务
   - DTO/Entity/VO分层清晰

需求规格：
[Paste Section 3-4 here]
```

---

## 使用说明

### 快速导航

| 角色 | 首要关注 | 次要关注 |
|------|----------|----------|
| **开发** | 第3章功能规格 + 第4章接口规格 | 第5章数据模型 + 第8章测试策略 |
| **测试** | 第3章AC场景 + 第6章错误处理 | 第5章测试数据 + 第8章测试清单 |
| **AI助手** | AI开发提示词模板 | 第3-4章具体用例 |

### 开发优先级

1. **Phase 1 (当前)**: P0用例 - 用户管理、题目识别、错因诊断、引导对话
2. **Phase 2**: P1用例 - 家长绑定、学习报告、知识图谱
3. **Phase 3**: 优化 - 性能提升、数据闭环、模型迭代
