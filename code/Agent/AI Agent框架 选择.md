> https://www.bilibili.com/video/BV1SqAVzLEci/?spm_id_from=333.1007.tianma.3-1-5.click&vd_source=b7200d0eaee914e9c128dcabce5df118

![[十个Agent框架.png]]

# 核心
## MCP 标准化
> MCP 全称 Model Context Protocol  大模型跟外部工具和数据源交互的标准协议；意味着一次封装，处处可用

## A2A协议 ：跨组织的Agent 交互
> A2A 全称 Agent to Agent Protocol  不同框架构建的Agent 怎么通信

MCP 解决的是Agent怎么用工具，A2A解决的是Agent和Agent之间怎么“说话”
目前原生支持A2A的Agent框架有：MAF（Microsoft Agent Framework） ，Google ADK ,AgentScope,
CrewAI 和 LangGraph 同感社区插件支持
## Context Engineering (上下文工程)
![[五大架构范式的差异化竞争.png]]

# Agent 框架的差异化竞争
## 第一种范式 ：图状态机
LangGraph : 少抽象，多控制
> LangGraph 提供 Node(节点),Edge（边），State(状态) 给开发者，LangGraph不预设Agent怎么思考，怎么协作；需要开发者自己设计状态模式，编排逻辑。错误处理

![[LangGraph 核心特性.png]]

## 角色驱动范式
CrewAI ：像组建真实团队一样组建Agent团队
> 每个 Agent 被赋予Role、Goal和Backstory

![[CrewAI.png]]
![[CrewAI适合干什么.png]]

## 事件驱动范式
Llama Index和AgentScope
Llama Index: 数据是Agent的基石
![[Llama Index.png]]
AgentScope:透明可控
![[AgentScope.png]]
## SDK封装 
OpenAI Agents SDK 和PydanticAI
![[OpenAI Agents SDK.png]]
PydanticAI :类型安全 把FastAPI的体验带给Agent开发
![[PydanticAI.png]]

## 第五种范式：低代码平台
Dify 
![[五大架构范式的特点.png]]

# 来自大厂的企业统一类框架
## Microsoft Agent Framework
> Microsoft Agent Framework 是由AutoGen 和Semantic Kernel 合并
> AutoGen 是学术界最前沿的多Agent研究框架
> Semantic Kernel 是企业级稳定的AI编排引擎

### MAF的优点
*  继承了AutoGen 的群聊、辩论、反思等先进多Agent 模式
* 原生支持.NET和python 的框架
* 原生支持A2A、AG-UI、MCP三大标准

## Google ADK 
> 多语言路线 ：python，ts ,java,go

![[Google ADK.png]]

# 选择框架
![[选择框架第一步.png]]![[选择框架第二步.png]]
![[未来.png]]