# PRD 转 WBS Excel 项目管理智能体 - 完整实现方案

## 1. 系统架构概述

本方案旨在构建一个自动化智能体，通过集成 TAPD 和 Confluence API 获取需求文档，利用大语言模型（LLM）进行语义理解和任务拆解（WBS），最终输出标准化的 Excel 格式 WBS 表格。系统采用前后端分离架构。

### 核心流程
1.  **前端交互**：用户在 Web 界面输入 TAPD 迭代ID 或 需求ID。
2.  **后端处理**：
    -   调用 TAPD API 获取需求详情（包含 Confluence 链接）。
    -   解析 Confluence 链接，调用 Confluence API 获取 PRD 文档全文。
3.  **智能分析**：
    -   后端将 PRD 文档内容发送给 LLM。
    -   LLM 基于预设的 WBS 拆解 Prompt 进行任务分解、工时估算和人员推荐。
4.  **结果预览与导出**：
    -   前端展示拆解后的 WBS 预览，支持用户微调。
    -   用户确认后，后端生成 Excel 文件供下载。

## 2. 技术栈选型

### 后端 (Backend)
-   **框架**: Node.js + NestJS (TypeScript)
-   **API 交互**: `axios` / `@nestjs/axios`
-   **大模型接口**: `openai` SDK 或 `langchain` (Node.js)
-   **Excel 处理**: `exceljs`
-   **文档解析**: `cheerio` (解析 HTML 内容)
-   **环境管理**: `@nestjs/config`

### 前端 (Frontend)
-   **框架**: Vue 3 (Composition API)
-   **UI 组件库**: Ant Design Vue
-   **HTTP 客户端**: Axios
-   **状态管理**: Pinia (可选)

## 3. 详细模块设计

### 3.1 后端模块 (NestJS Modules)

#### 1. TapdModule
负责获取迭代下的需求列表。
-   **Service**: `TapdService`
    -   `getStory(storyId)`: 获取单个需求详情。
    -   `getIterationStories(iterationId)`: 获取迭代下的所有需求。

#### 2. ConfluenceModule
负责获取 PRD 详细内容。
-   **Service**: `ConfluenceService`
    -   `getPageContent(pageId)`: 获取 Wiki 页面内容。
    -   `cleanHtml(html)`: 使用 `cheerio` 提取纯文本，去除 HTML 标签。

#### 3. AiModule
负责与 LLM 交互。
-   **Service**: `AiService`
    -   `generateWbs(content)`: 调用 LLM 生成 WBS JSON 数据。
    -   **Prompt Management**: 维护系统提示词模板。

#### 4. ExcelModule
负责生成 Excel 文件。
-   **Service**: `ExcelService`
    -   `generateExcel(data)`: 将 JSON 数据转换为 Excel Buffer 流。

#### 5. WbsModule (Main Application Logic)
协调上述服务，处理业务逻辑。
-   **Controller**: `WbsController`
    -   `POST /wbs/analyze`: 接收 ID，返回预览数据。
    -   `POST /wbs/export`: 接收确认后的数据，返回 Excel 文件流。

### 3.2 前端模块 (Vue 3 Components)

#### 1. InputPanel
-   输入框：TAPD ID / Iteration ID。
-   配置项：选择开发人员名单（可选）。

#### 2. WbsPreviewTable
-   使用 Ant Design Table 展示拆解结果。
-   支持行编辑：修改工时、负责人、任务描述。

#### 3. ActionToolbar
-   按钮：开始分析、导出 Excel。

## 4. 数据结构与 Prompt 设计

### Prompt 结构示例
```markdown
角色：你是一位资深的项目经理和系统架构师。
任务：根据提供的 PRD 文档内容，进行详细的 WBS（工作分解结构）拆解。

输入文档：
{PRD_CONTENT}

输出要求：
请输出 JSON 格式，包含以下字段：
- iteration_name: 迭代名称
- tapd_id: 需求ID
- tasks: 任务列表
  - task_name: 子任务名称
  - task_desc: 详细描述
  - developer: 推荐开发角色（前端/后端/测试）
  - estimate: 预估工时（小时）

拆解原则：
1. 任务粒度控制在 4-8 小时之间。
2. 包含前端、后端、联调、测试等环节。
```

## 5. 实现步骤与代码结构

### 目录结构 (Monorepo 风格推荐)
```
project-manager-agent/
├── backend/                # NestJS 项目
│   ├── src/
│   │   ├── app.module.ts
│   │   ├── tapd/
│   │   ├── confluence/
│   │   ├── ai/
│   │   ├── excel/
│   │   └── wbs/
│   ├── .env
│   └── package.json
├── frontend/               # Vue 3 项目
│   ├── src/
│   │   ├── components/
│   │   ├── App.vue
│   │   └── main.ts
│   └── package.json
└── README.md
```

### 关键代码逻辑 (后端 TypeScript)

#### 1. Confluence 内容获取 (NestJS Service)
```typescript
import { HttpService } from '@nestjs/axios';
import * as cheerio from 'cheerio';

@Injectable()
export class ConfluenceService {
  constructor(private readonly httpService: HttpService) {}

  async getPageContent(pageId: string): Promise<string> {
    const url = `${process.env.CONF_BASE_URL}/wiki/rest/api/content/${pageId}?expand=body.storage`;
    const { data } = await this.httpService.axiosRef.get(url, {
      auth: { username: process.env.CONF_USER, password: process.env.CONF_TOKEN }
    });
    
    const html = data.body.storage.value;
    const $ = cheerio.load(html);
    return $.text(); // 提取纯文本
  }
}
```

#### 2. LLM 调用 (OpenAI SDK)
```typescript
import OpenAI from 'openai';

@Injectable()
export class AiService {
  private openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

  async generateWbs(content: string): Promise<any> {
    const prompt = `...Prompt Template... ${content}`;
    const completion = await this.openai.chat.completions.create({
      messages: [{ role: 'user', content: prompt }],
      model: 'gpt-4',
      response_format: { type: 'json_object' },
    });
    
    return JSON.parse(completion.choices[0].message.content);
  }
}
```

#### 3. Excel 生成 (ExcelJS)
```typescript
import * as ExcelJS from 'exceljs';

@Injectable()
export class ExcelService {
  async createExcel(wbsData: any): Promise<Buffer> {
    const workbook = new ExcelJS.Workbook();
    const sheet = workbook.addWorksheet('WBS');
    
    sheet.columns = [
      { header: '迭代名称', key: 'iteration', width: 20 },
      { header: '需求ID', key: 'tapd_id', width: 15 },
      { header: '任务名称', key: 'task_name', width: 30 },
      { header: '负责人', key: 'developer', width: 15 },
      { header: '工时', key: 'estimate', width: 10 },
    ];
    
    wbsData.tasks.forEach(task => {
      sheet.addRow({
        iteration: wbsData.iteration_name,
        tapd_id: wbsData.tapd_id,
        task_name: task.task_name,
        developer: task.developer,
        estimate: task.estimate
      });
    });

    return await workbook.xlsx.writeBuffer() as Buffer;
  }
}
```

## 6. 扩展功能建议

1.  **人员技能矩阵配置**：在前端设置页面配置团队成员及其擅长领域，传给后端作为 Prompt 的上下文。
2.  **多轮对话调整**：如果初次拆解不满意，用户可以在前端输入修改意见，后端调用 LLM 进行 Refine（优化）。
3.  **Webhook 集成**：当 TAPD 需求状态变更时，自动触发分析流程。
