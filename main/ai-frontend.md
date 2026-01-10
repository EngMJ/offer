# AI 前端集成面试题

> 本文档涵盖 LLM 应用前端架构、API 集成、Streaming 处理、RAG 应用等 AI 前端面试题

---

## 一、AI 前端基础

### 1. 前端如何与 LLM 服务集成？

**架构模式：**

```
┌─────────────────────────────────────────────────────────────┐
│                        前端应用                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              AI 对话组件                              │   │
│  │  输入框 → 消息列表 → Markdown 渲染 → 代码高亮         │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────┬───────────────────────────────┘
                              │ SSE / WebSocket
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      BFF / API 层                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  认证 → 限流 → Prompt 模板 → 上下文管理 → 流式转发   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────┬───────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│   OpenAI API  │    │  Claude API   │    │  私有部署 LLM │
└───────────────┘    └───────────────┘    └───────────────┘
```

**基本集成示例：**

```typescript
// api/chat.ts
interface ChatMessage {
  role: 'user' | 'assistant' | 'system';
  content: string;
}

interface ChatRequest {
  messages: ChatMessage[];
  model?: string;
  temperature?: number;
  max_tokens?: number;
  stream?: boolean;
}

// 非流式调用
async function chat(request: ChatRequest): Promise<string> {
  const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(request),
  });
  
  if (!response.ok) {
    throw new Error(`Chat failed: ${response.statusText}`);
  }
  
  const data = await response.json();
  return data.content;
}

// 流式调用
async function* chatStream(request: ChatRequest): AsyncGenerator<string> {
  const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ ...request, stream: true }),
  });
  
  if (!response.ok) {
    throw new Error(`Chat failed: ${response.statusText}`);
  }
  
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    const chunk = decoder.decode(value, { stream: true });
    yield chunk;
  }
}
```

---

### 2. 如何处理 SSE (Server-Sent Events) 流式响应？

```typescript
// 使用 EventSource（简单场景）
function useSSE(url: string) {
  const [data, setData] = useState<string>('');
  const [error, setError] = useState<Error | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const start = useCallback((body: any) => {
    setIsLoading(true);
    setData('');
    
    // EventSource 不支持 POST，需要使用 fetch
    fetchSSE(url, body, {
      onMessage: (chunk) => {
        setData(prev => prev + chunk);
      },
      onError: (err) => {
        setError(err);
        setIsLoading(false);
      },
      onComplete: () => {
        setIsLoading(false);
      },
    });
  }, [url]);
  
  return { data, error, isLoading, start };
}

// SSE 解析器
async function fetchSSE(
  url: string,
  body: any,
  callbacks: {
    onMessage: (chunk: string) => void;
    onError: (error: Error) => void;
    onComplete: () => void;
  }
) {
  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'text/event-stream',
      },
      body: JSON.stringify(body),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    
    const reader = response.body!.getReader();
    const decoder = new TextDecoder();
    let buffer = '';
    
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      
      buffer += decoder.decode(value, { stream: true });
      
      // 解析 SSE 格式
      const lines = buffer.split('\n');
      buffer = lines.pop() || '';
      
      for (const line of lines) {
        if (line.startsWith('data: ')) {
          const data = line.slice(6);
          
          if (data === '[DONE]') {
            callbacks.onComplete();
            return;
          }
          
          try {
            const parsed = JSON.parse(data);
            const content = parsed.choices?.[0]?.delta?.content;
            if (content) {
              callbacks.onMessage(content);
            }
          } catch {
            // 非 JSON 数据，直接输出
            callbacks.onMessage(data);
          }
        }
      }
    }
    
    callbacks.onComplete();
  } catch (error) {
    callbacks.onError(error as Error);
  }
}
```

---

### 3. 如何使用 ReadableStream 处理流式数据？

```typescript
// 更底层的 ReadableStream 处理
async function processStream(response: Response) {
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  
  // 创建一个可读流来处理数据
  const stream = new ReadableStream({
    async start(controller) {
      while (true) {
        const { done, value } = await reader.read();
        
        if (done) {
          controller.close();
          break;
        }
        
        const text = decoder.decode(value, { stream: true });
        controller.enqueue(text);
      }
    },
  });
  
  return stream;
}

// 使用 TransformStream 进行数据转换
function createSSEParser() {
  let buffer = '';
  
  return new TransformStream({
    transform(chunk, controller) {
      buffer += chunk;
      const lines = buffer.split('\n\n');
      buffer = lines.pop() || '';
      
      for (const event of lines) {
        if (event.startsWith('data: ')) {
          const data = event.slice(6);
          if (data !== '[DONE]') {
            try {
              controller.enqueue(JSON.parse(data));
            } catch {
              controller.enqueue({ content: data });
            }
          }
        }
      }
    },
    flush(controller) {
      if (buffer) {
        controller.enqueue({ content: buffer });
      }
    },
  });
}

// 组合使用
async function streamChat(messages: ChatMessage[]) {
  const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ messages, stream: true }),
  });
  
  const stream = response.body!
    .pipeThrough(new TextDecoderStream())
    .pipeThrough(createSSEParser());
  
  const reader = stream.getReader();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    console.log('Received:', value);
  }
}
```

---

## 二、AI 对话组件设计

### 4. 如何设计 AI 对话组件？

```tsx
// components/ChatInterface.tsx
import { useState, useRef, useEffect } from 'react';
import { useChatStream } from '@/hooks/useChatStream';
import { MessageList } from './MessageList';
import { MessageInput } from './MessageInput';

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
  status?: 'sending' | 'streaming' | 'complete' | 'error';
}

export function ChatInterface() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);
  
  const { streamMessage, isStreaming, abort } = useChatStream({
    onChunk: (chunk, messageId) => {
      setMessages(prev => prev.map(msg =>
        msg.id === messageId
          ? { ...msg, content: msg.content + chunk }
          : msg
      ));
    },
    onComplete: (messageId) => {
      setMessages(prev => prev.map(msg =>
        msg.id === messageId
          ? { ...msg, status: 'complete' }
          : msg
      ));
    },
    onError: (error, messageId) => {
      setMessages(prev => prev.map(msg =>
        msg.id === messageId
          ? { ...msg, status: 'error', content: `Error: ${error.message}` }
          : msg
      ));
    },
  });
  
  // 自动滚动到底部
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);
  
  const handleSend = async () => {
    if (!input.trim() || isStreaming) return;
    
    const userMessage: Message = {
      id: generateId(),
      role: 'user',
      content: input,
      timestamp: new Date(),
      status: 'complete',
    };
    
    const assistantMessage: Message = {
      id: generateId(),
      role: 'assistant',
      content: '',
      timestamp: new Date(),
      status: 'streaming',
    };
    
    setMessages(prev => [...prev, userMessage, assistantMessage]);
    setInput('');
    
    await streamMessage(
      [...messages, userMessage].map(m => ({
        role: m.role,
        content: m.content,
      })),
      assistantMessage.id
    );
  };
  
  return (
    <div className="chat-container">
      <MessageList messages={messages} />
      <div ref={messagesEndRef} />
      
      <MessageInput
        value={input}
        onChange={setInput}
        onSend={handleSend}
        isStreaming={isStreaming}
        onAbort={abort}
      />
    </div>
  );
}

// hooks/useChatStream.ts
export function useChatStream(callbacks: {
  onChunk: (chunk: string, messageId: string) => void;
  onComplete: (messageId: string) => void;
  onError: (error: Error, messageId: string) => void;
}) {
  const [isStreaming, setIsStreaming] = useState(false);
  const abortControllerRef = useRef<AbortController | null>(null);
  
  const streamMessage = useCallback(async (
    messages: { role: string; content: string }[],
    messageId: string
  ) => {
    abortControllerRef.current = new AbortController();
    setIsStreaming(true);
    
    try {
      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages, stream: true }),
        signal: abortControllerRef.current.signal,
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      const reader = response.body!.getReader();
      const decoder = new TextDecoder();
      
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        const chunk = decoder.decode(value, { stream: true });
        // 解析 SSE 数据
        const lines = chunk.split('\n');
        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = line.slice(6);
            if (data === '[DONE]') continue;
            try {
              const parsed = JSON.parse(data);
              const content = parsed.choices?.[0]?.delta?.content;
              if (content) {
                callbacks.onChunk(content, messageId);
              }
            } catch {}
          }
        }
      }
      
      callbacks.onComplete(messageId);
    } catch (error) {
      if ((error as Error).name !== 'AbortError') {
        callbacks.onError(error as Error, messageId);
      }
    } finally {
      setIsStreaming(false);
      abortControllerRef.current = null;
    }
  }, [callbacks]);
  
  const abort = useCallback(() => {
    abortControllerRef.current?.abort();
  }, []);
  
  return { streamMessage, isStreaming, abort };
}
```

---

### 5. 如何渲染 AI 生成的 Markdown 内容？

```tsx
// components/MarkdownRenderer.tsx
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import remarkMath from 'remark-math';
import rehypeKatex from 'rehype-katex';
import rehypeHighlight from 'rehype-highlight';
import { useState } from 'react';

interface MarkdownRendererProps {
  content: string;
  isStreaming?: boolean;
}

export function MarkdownRenderer({ content, isStreaming }: MarkdownRendererProps) {
  return (
    <div className="markdown-body">
      <ReactMarkdown
        remarkPlugins={[remarkGfm, remarkMath]}
        rehypePlugins={[rehypeKatex, rehypeHighlight]}
        components={{
          // 自定义代码块
          code({ node, inline, className, children, ...props }) {
            const match = /language-(\w+)/.exec(className || '');
            const language = match ? match[1] : '';
            
            if (inline) {
              return (
                <code className="inline-code" {...props}>
                  {children}
                </code>
              );
            }
            
            return (
              <CodeBlock
                language={language}
                code={String(children).replace(/\n$/, '')}
              />
            );
          },
          // 自定义链接
          a({ href, children }) {
            return (
              <a href={href} target="_blank" rel="noopener noreferrer">
                {children}
              </a>
            );
          },
          // 自定义表格
          table({ children }) {
            return (
              <div className="table-wrapper">
                <table>{children}</table>
              </div>
            );
          },
        }}
      >
        {content}
      </ReactMarkdown>
      
      {/* 流式输出时显示光标 */}
      {isStreaming && <span className="streaming-cursor">▊</span>}
    </div>
  );
}

// components/CodeBlock.tsx
function CodeBlock({ language, code }: { language: string; code: string }) {
  const [copied, setCopied] = useState(false);
  
  const handleCopy = async () => {
    await navigator.clipboard.writeText(code);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };
  
  return (
    <div className="code-block">
      <div className="code-header">
        <span className="language">{language}</span>
        <button onClick={handleCopy} className="copy-btn">
          {copied ? '已复制' : '复制'}
        </button>
      </div>
      <pre>
        <code className={`language-${language}`}>{code}</code>
      </pre>
    </div>
  );
}
```

---

## 三、Prompt 工程

### 6. 前端如何管理和优化 Prompt？

```typescript
// prompts/templates.ts
interface PromptTemplate {
  name: string;
  system: string;
  user: string;
  variables: string[];
}

const promptTemplates: Record<string, PromptTemplate> = {
  codeReview: {
    name: '代码审查',
    system: `你是一个资深的代码审查专家。请审查用户提供的代码，关注以下方面：
1. 代码质量和可读性
2. 潜在的 Bug 和安全问题
3. 性能优化建议
4. 最佳实践建议

请用中文回复，并提供具体的改进建议。`,
    user: '请审查以下 {{language}} 代码：\n\n```{{language}}\n{{code}}\n```',
    variables: ['language', 'code'],
  },
  
  translate: {
    name: '翻译',
    system: '你是一个专业的翻译专家。请将用户提供的文本翻译成{{targetLanguage}}，保持原文的语气和风格。',
    user: '{{text}}',
    variables: ['targetLanguage', 'text'],
  },
  
  summarize: {
    name: '文章摘要',
    system: '你是一个文章摘要专家。请用简洁的语言总结文章的主要观点，控制在{{maxLength}}字以内。',
    user: '请总结以下文章：\n\n{{article}}',
    variables: ['maxLength', 'article'],
  },
};

// 模板渲染函数
function renderTemplate(
  template: PromptTemplate,
  variables: Record<string, string>
): { system: string; user: string } {
  let system = template.system;
  let user = template.user;
  
  for (const [key, value] of Object.entries(variables)) {
    const placeholder = new RegExp(`{{${key}}}`, 'g');
    system = system.replace(placeholder, value);
    user = user.replace(placeholder, value);
  }
  
  return { system, user };
}

// 使用示例
const { system, user } = renderTemplate(promptTemplates.codeReview, {
  language: 'TypeScript',
  code: 'function add(a, b) { return a + b }',
});

// Prompt 管理 Hook
function usePromptManager() {
  const [history, setHistory] = useState<PromptHistory[]>([]);
  
  const savePrompt = (prompt: { system: string; user: string; response: string }) => {
    const entry: PromptHistory = {
      id: generateId(),
      ...prompt,
      timestamp: new Date(),
      tokens: estimateTokens(prompt.system + prompt.user + prompt.response),
    };
    
    setHistory(prev => [entry, ...prev].slice(0, 100)); // 保留最近 100 条
    localStorage.setItem('prompt_history', JSON.stringify(history));
  };
  
  return { history, savePrompt };
}
```

---

### 7. 如何实现 Prompt 的 A/B 测试？

```typescript
// services/promptABTest.ts
interface PromptVariant {
  id: string;
  name: string;
  systemPrompt: string;
  weight: number; // 流量权重
}

interface ABTestConfig {
  id: string;
  name: string;
  variants: PromptVariant[];
  metrics: string[];
}

class PromptABTest {
  private config: ABTestConfig;
  private userId: string;
  
  constructor(config: ABTestConfig, userId: string) {
    this.config = config;
    this.userId = userId;
  }
  
  // 基于用户 ID 确定性分组
  getVariant(): PromptVariant {
    const hash = this.hashString(this.userId + this.config.id);
    const normalizedHash = hash / 0xFFFFFFFF;
    
    let cumulativeWeight = 0;
    for (const variant of this.config.variants) {
      cumulativeWeight += variant.weight;
      if (normalizedHash < cumulativeWeight) {
        return variant;
      }
    }
    
    return this.config.variants[0];
  }
  
  // 记录实验数据
  trackMetric(metricName: string, value: number) {
    const variant = this.getVariant();
    
    analytics.track('prompt_ab_test', {
      testId: this.config.id,
      variantId: variant.id,
      metricName,
      value,
      userId: this.userId,
    });
  }
  
  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) - hash) + str.charCodeAt(i);
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
}

// 使用示例
const abTest = new PromptABTest({
  id: 'chat_system_prompt_v2',
  name: '对话系统提示词优化',
  variants: [
    {
      id: 'control',
      name: '对照组',
      systemPrompt: '你是一个有帮助的助手。',
      weight: 0.5,
    },
    {
      id: 'treatment',
      name: '实验组',
      systemPrompt: '你是一个专业、友好的 AI 助手。请用简洁清晰的语言回答问题。',
      weight: 0.5,
    },
  ],
  metrics: ['satisfaction_rate', 'retry_rate', 'avg_turns'],
}, userId);

const variant = abTest.getVariant();
// 使用 variant.systemPrompt 进行对话

// 记录用户满意度
abTest.trackMetric('satisfaction_rate', userRating);
```

---

## 四、RAG 应用

### 8. 什么是 RAG？前端如何实现？

**RAG (Retrieval-Augmented Generation)** 是结合检索和生成的 AI 应用模式。

```
用户问题 → 向量化 → 相似度检索 → 获取相关文档 → 构建 Prompt → LLM 生成
```

```typescript
// services/ragService.ts
interface Document {
  id: string;
  content: string;
  metadata: {
    source: string;
    title: string;
    [key: string]: any;
  };
}

interface RAGRequest {
  query: string;
  topK?: number;
  filters?: Record<string, any>;
}

interface RAGResponse {
  answer: string;
  sources: Document[];
  confidence: number;
}

async function ragQuery(request: RAGRequest): Promise<RAGResponse> {
  // 1. 检索相关文档
  const searchResponse = await fetch('/api/search', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      query: request.query,
      topK: request.topK || 5,
      filters: request.filters,
    }),
  });
  
  const { documents } = await searchResponse.json();
  
  // 2. 构建带上下文的 Prompt
  const context = documents
    .map((doc: Document) => `[来源: ${doc.metadata.title}]\n${doc.content}`)
    .join('\n\n---\n\n');
  
  const systemPrompt = `你是一个知识库助手。请基于以下参考资料回答用户的问题。
如果参考资料中没有相关信息，请明确告知用户。
请在回答末尾标注引用的来源。

参考资料：
${context}`;
  
  // 3. 调用 LLM
  const chatResponse = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: request.query },
      ],
      stream: false,
    }),
  });
  
  const { content } = await chatResponse.json();
  
  return {
    answer: content,
    sources: documents,
    confidence: calculateConfidence(documents),
  };
}

// RAG 聊天 Hook
function useRAGChat() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [sources, setSources] = useState<Document[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  
  const ask = async (query: string) => {
    setIsLoading(true);
    
    const userMessage = { role: 'user', content: query };
    setMessages(prev => [...prev, userMessage]);
    
    try {
      const response = await ragQuery({ query });
      
      const assistantMessage = {
        role: 'assistant',
        content: response.answer,
        sources: response.sources,
      };
      
      setMessages(prev => [...prev, assistantMessage]);
      setSources(response.sources);
    } catch (error) {
      console.error('RAG query failed:', error);
    } finally {
      setIsLoading(false);
    }
  };
  
  return { messages, sources, isLoading, ask };
}
```

---

### 9. 如何实现文档上传和向量化？

```tsx
// components/DocumentUploader.tsx
import { useState, useCallback } from 'react';

interface UploadedDocument {
  id: string;
  filename: string;
  status: 'uploading' | 'processing' | 'indexed' | 'error';
  progress?: number;
  error?: string;
}

export function DocumentUploader() {
  const [documents, setDocuments] = useState<UploadedDocument[]>([]);
  
  const handleUpload = useCallback(async (files: FileList) => {
    for (const file of Array.from(files)) {
      const docId = generateId();
      
      // 添加到列表
      setDocuments(prev => [...prev, {
        id: docId,
        filename: file.name,
        status: 'uploading',
        progress: 0,
      }]);
      
      try {
        // 1. 上传文件
        const formData = new FormData();
        formData.append('file', file);
        
        const uploadResponse = await fetch('/api/documents/upload', {
          method: 'POST',
          body: formData,
        });
        
        if (!uploadResponse.ok) {
          throw new Error('Upload failed');
        }
        
        const { documentId } = await uploadResponse.json();
        
        // 更新状态为处理中
        setDocuments(prev => prev.map(doc =>
          doc.id === docId ? { ...doc, status: 'processing' } : doc
        ));
        
        // 2. 触发向量化处理
        const processResponse = await fetch('/api/documents/process', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ documentId }),
        });
        
        if (!processResponse.ok) {
          throw new Error('Processing failed');
        }
        
        // 3. 轮询处理状态
        await pollProcessingStatus(documentId, (progress) => {
          setDocuments(prev => prev.map(doc =>
            doc.id === docId ? { ...doc, progress } : doc
          ));
        });
        
        // 更新状态为已索引
        setDocuments(prev => prev.map(doc =>
          doc.id === docId ? { ...doc, status: 'indexed', progress: 100 } : doc
        ));
        
      } catch (error) {
        setDocuments(prev => prev.map(doc =>
          doc.id === docId
            ? { ...doc, status: 'error', error: (error as Error).message }
            : doc
        ));
      }
    }
  }, []);
  
  return (
    <div className="document-uploader">
      <input
        type="file"
        multiple
        accept=".pdf,.txt,.md,.docx"
        onChange={(e) => e.target.files && handleUpload(e.target.files)}
      />
      
      <ul className="document-list">
        {documents.map(doc => (
          <li key={doc.id} className={`status-${doc.status}`}>
            <span>{doc.filename}</span>
            {doc.status === 'uploading' && <progress value={doc.progress} max={100} />}
            {doc.status === 'processing' && <span>处理中...</span>}
            {doc.status === 'indexed' && <span>✓ 已索引</span>}
            {doc.status === 'error' && <span>✗ {doc.error}</span>}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 五、Token 管理

### 10. 如何估算和优化 Token 消耗？

```typescript
// utils/tokenEstimator.ts
// 简单的 Token 估算（实际应使用 tiktoken）
function estimateTokens(text: string): number {
  // 英文约 4 字符 = 1 token
  // 中文约 2 字符 = 1 token
  const englishChars = text.replace(/[\u4e00-\u9fa5]/g, '').length;
  const chineseChars = (text.match(/[\u4e00-\u9fa5]/g) || []).length;
  
  return Math.ceil(englishChars / 4 + chineseChars / 2);
}

// Token 限制管理
interface TokenBudget {
  maxInputTokens: number;
  maxOutputTokens: number;
  reservedTokens: number; // 为系统提示预留
}

function truncateMessages(
  messages: ChatMessage[],
  budget: TokenBudget
): ChatMessage[] {
  const maxTokens = budget.maxInputTokens - budget.reservedTokens;
  let totalTokens = 0;
  const result: ChatMessage[] = [];
  
  // 从最新消息开始保留
  for (let i = messages.length - 1; i >= 0; i--) {
    const tokens = estimateTokens(messages[i].content);
    
    if (totalTokens + tokens > maxTokens) {
      break;
    }
    
    totalTokens += tokens;
    result.unshift(messages[i]);
  }
  
  return result;
}

// Token 使用追踪
class TokenTracker {
  private usage: Map<string, number> = new Map();
  private dailyLimit: number;
  
  constructor(dailyLimit: number = 100000) {
    this.dailyLimit = dailyLimit;
    this.loadFromStorage();
  }
  
  track(tokens: number, userId: string) {
    const today = new Date().toDateString();
    const key = `${userId}:${today}`;
    
    const current = this.usage.get(key) || 0;
    this.usage.set(key, current + tokens);
    
    this.saveToStorage();
  }
  
  getRemainingTokens(userId: string): number {
    const today = new Date().toDateString();
    const key = `${userId}:${today}`;
    
    const used = this.usage.get(key) || 0;
    return Math.max(0, this.dailyLimit - used);
  }
  
  canUseTokens(userId: string, required: number): boolean {
    return this.getRemainingTokens(userId) >= required;
  }
  
  private loadFromStorage() {
    const saved = localStorage.getItem('token_usage');
    if (saved) {
      this.usage = new Map(JSON.parse(saved));
    }
  }
  
  private saveToStorage() {
    localStorage.setItem('token_usage', JSON.stringify([...this.usage]));
  }
}

export const tokenTracker = new TokenTracker();
```

---

## 六、多模态交互

### 11. 如何实现图像输入的 AI 对话？

```tsx
// components/MultimodalChat.tsx
import { useState, useCallback } from 'react';

interface ImageContent {
  type: 'image_url';
  image_url: {
    url: string;
    detail?: 'low' | 'high' | 'auto';
  };
}

interface TextContent {
  type: 'text';
  text: string;
}

type MessageContent = string | (TextContent | ImageContent)[];

interface MultimodalMessage {
  role: 'user' | 'assistant';
  content: MessageContent;
}

export function MultimodalChat() {
  const [messages, setMessages] = useState<MultimodalMessage[]>([]);
  const [images, setImages] = useState<string[]>([]);
  const [input, setInput] = useState('');
  
  // 图片转 Base64
  const handleImageSelect = useCallback(async (file: File) => {
    const reader = new FileReader();
    
    return new Promise<string>((resolve) => {
      reader.onload = () => {
        const base64 = reader.result as string;
        resolve(base64);
      };
      reader.readAsDataURL(file);
    });
  }, []);
  
  const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const files = e.target.files;
    if (!files) return;
    
    const newImages = await Promise.all(
      Array.from(files).map(handleImageSelect)
    );
    
    setImages(prev => [...prev, ...newImages]);
  };
  
  const handleSend = async () => {
    if (!input.trim() && images.length === 0) return;
    
    // 构建多模态消息内容
    const content: (TextContent | ImageContent)[] = [];
    
    // 添加图片
    for (const imageUrl of images) {
      content.push({
        type: 'image_url',
        image_url: {
          url: imageUrl,
          detail: 'auto',
        },
      });
    }
    
    // 添加文本
    if (input.trim()) {
      content.push({
        type: 'text',
        text: input,
      });
    }
    
    const userMessage: MultimodalMessage = {
      role: 'user',
      content,
    };
    
    setMessages(prev => [...prev, userMessage]);
    setImages([]);
    setInput('');
    
    // 调用多模态 API
    const response = await fetch('/api/chat/vision', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: 'gpt-4-vision-preview',
        messages: [...messages, userMessage],
        max_tokens: 1000,
      }),
    });
    
    const data = await response.json();
    
    setMessages(prev => [...prev, {
      role: 'assistant',
      content: data.content,
    }]);
  };
  
  return (
    <div className="multimodal-chat">
      <MessageList messages={messages} />
      
      {/* 图片预览 */}
      <div className="image-preview">
        {images.map((img, idx) => (
          <div key={idx} className="preview-item">
            <img src={img} alt={`Preview ${idx}`} />
            <button onClick={() => setImages(prev => prev.filter((_, i) => i !== idx))}>
              ×
            </button>
          </div>
        ))}
      </div>
      
      <div className="input-area">
        <input
          type="file"
          accept="image/*"
          multiple
          onChange={handleFileChange}
        />
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="描述图片或提问..."
        />
        <button onClick={handleSend}>发送</button>
      </div>
    </div>
  );
}
```

---

## 七、安全与最佳实践

### 12. AI 前端应用有哪些安全注意事项？

```typescript
// 1. API Key 保护 - 永远不要在前端暴露
// ❌ 错误做法
const response = await fetch('https://api.openai.com/v1/chat/completions', {
  headers: {
    'Authorization': `Bearer ${OPENAI_API_KEY}`, // 危险！
  },
});

// ✅ 正确做法 - 通过后端代理
const response = await fetch('/api/chat', {
  headers: {
    'Authorization': `Bearer ${userToken}`, // 用户 Token
  },
});

// 2. 输入验证和清理
function sanitizeUserInput(input: string): string {
  // 移除潜在的注入攻击内容
  return input
    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
    .replace(/<!--[\s\S]*?-->/g, '')
    .trim()
    .slice(0, 10000); // 限制长度
}

// 3. Prompt 注入防护
function buildSafePrompt(systemPrompt: string, userInput: string): string {
  // 将用户输入用明确的分隔符包裹
  return `${systemPrompt}

用户消息（以下内容来自用户输入，请谨慎处理）：
"""
${sanitizeUserInput(userInput)}
"""`;
}

// 4. 速率限制（客户端）
class RateLimiter {
  private tokens: number;
  private lastRefill: number;
  private maxTokens: number;
  private refillRate: number; // tokens per second
  
  constructor(maxTokens: number, refillRate: number) {
    this.tokens = maxTokens;
    this.lastRefill = Date.now();
    this.maxTokens = maxTokens;
    this.refillRate = refillRate;
  }
  
  canMakeRequest(): boolean {
    this.refill();
    return this.tokens > 0;
  }
  
  consumeToken(): boolean {
    if (this.canMakeRequest()) {
      this.tokens--;
      return true;
    }
    return false;
  }
  
  private refill() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    this.tokens = Math.min(this.maxTokens, this.tokens + elapsed * this.refillRate);
    this.lastRefill = now;
  }
}

// 5. 内容过滤
async function moderateContent(content: string): Promise<boolean> {
  const response = await fetch('/api/moderation', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ content }),
  });
  
  const { flagged, categories } = await response.json();
  
  if (flagged) {
    console.warn('Content flagged:', categories);
    return false;
  }
  
  return true;
}

// 6. 错误处理和降级
async function chatWithFallback(messages: ChatMessage[]) {
  try {
    return await chat(messages);
  } catch (error) {
    // API 错误，使用降级响应
    if (error.status === 429) {
      return '当前请求过多，请稍后再试。';
    }
    if (error.status === 503) {
      return 'AI 服务暂时不可用，请稍后再试。';
    }
    throw error;
  }
}
```

---

### 13. AI 应用的性能优化有哪些方法？

```typescript
// 1. 响应缓存
const responseCache = new Map<string, { content: string; timestamp: number }>();
const CACHE_TTL = 5 * 60 * 1000; // 5 分钟

async function cachedChat(messages: ChatMessage[]): Promise<string> {
  const cacheKey = JSON.stringify(messages);
  const cached = responseCache.get(cacheKey);
  
  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.content;
  }
  
  const content = await chat(messages);
  responseCache.set(cacheKey, { content, timestamp: Date.now() });
  
  return content;
}

// 2. 流式渲染优化 - 使用 requestAnimationFrame 批量更新
function useOptimizedStream() {
  const [content, setContent] = useState('');
  const pendingChunksRef = useRef<string[]>([]);
  const rafIdRef = useRef<number>();
  
  const appendChunk = useCallback((chunk: string) => {
    pendingChunksRef.current.push(chunk);
    
    if (!rafIdRef.current) {
      rafIdRef.current = requestAnimationFrame(() => {
        setContent(prev => prev + pendingChunksRef.current.join(''));
        pendingChunksRef.current = [];
        rafIdRef.current = undefined;
      });
    }
  }, []);
  
  useEffect(() => {
    return () => {
      if (rafIdRef.current) {
        cancelAnimationFrame(rafIdRef.current);
      }
    };
  }, []);
  
  return { content, appendChunk };
}

// 3. 虚拟滚动 - 长对话列表优化
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualMessageList({ messages }: { messages: Message[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: messages.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100, // 估计每条消息高度
    overscan: 5,
  });
  
  return (
    <div ref={parentRef} style={{ height: '100%', overflow: 'auto' }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            <MessageItem message={messages[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}

// 4. 预加载和预测
function usePredictiveLoading() {
  const [isTyping, setIsTyping] = useState(false);
  
  // 当用户开始输入时，预热连接
  useEffect(() => {
    if (isTyping) {
      // 发送一个预热请求
      fetch('/api/chat/warmup', { method: 'HEAD' });
    }
  }, [isTyping]);
  
  return { setIsTyping };
}
```

---

