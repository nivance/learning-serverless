---
date: 2018-10-24T23:10:00+08:00
title: 核心规范Version 0.1
weight: 212
menu:
  main:
    parent: "spec-cloudevents"
description : "CloudEvents核心规范中文翻译版本"
---

> 备注：内容来自　https://github.com/cloudevents/spec/blob/v0.1/spec.md

## 摘要

CloudEvents是一种供应商中立的规范，用于定义事件数据的格式。

## 文档状态

本文档是一份工作草案。

## 概述

事件无处不在。 但是，事件发布者倾向于以不同方式描述事件。

缺乏描述事件的常用方法意味着开发人员必须不断重新学习如何接收事件。这也限制了库，工具和基础设施的可能性，以帮助跨环境（如SDK，事件路由器或跟踪系统）发送事件数据。我们可以从事件数据中实现的可移植性和生产力总体上受到阻碍。

来到CloudEvents，这是一种以通用方式描述事件数据的规范。CloudEvents旨在简化跨服务，平台及其他方面的事件声明和发送。

事件格式指定如何使用某些编码格式序列化CloudEvent。支持这些编码的兼容CloudEvents实现必须遵守相应事件格式中指定的编码规则。所有实现必须支持JSON格式。

## 设计目标

CloudEvents通常用于分布式系统，以允许服务在开发期间松散耦合，独立部署，以后可以连接以创建新应用程序。

CloudEvents规范的目标是定义事件系统的互操作性，这些事件系统允许服务生成或消费事件，其中生产者和消费者可以独立开发和部署。生产者可以在消费者收听之前生成事件，并且消费者可以表达对尚未生成的事件或事件类别的兴趣。

为此，规范将包括促进互操作性的事件的公共元数据属性，而事件不包含可能用于发送事件的消费者或传输的任何细节。

### 非目标

以下内容不在规范中：

- 函数构建和调用过程
- 特定于语言的运行时API
- 选择单一身份/访问控制系统

## 使用场景

下面的列表列举了为开发此规范而考虑的关键使用场景和开发人员视角。这些使用场景并非详尽无遗，并且该规范并非旨在规定如何使用。

这些情景不是规范性的; 任何人都可以自由地创建一个混合这些场景的系统。 这些案例建立了事件生成器，消费者，中间件和框架的通用词汇表。

在这些场景中，我们将事件生成器和事件使用者的角色保持不同。 单个应用程序上下文始终可以同时承担多个角色，包括既是事件生产者又是事件消费者。

1. 应用程序生成供其他方使用的事件，例如为消费者提供有关终端用户活动，状态更改或环境观察的见解，或允许通过事件驱动的扩展来补充应用程序的功能。

	通常生成与上下文或生产者选择的分类相关的事件。例如，房间中的温度传感器可能由安装位置，房间，楼层和建筑物进行上下文限定。运动会结果可能按联赛和球队分类。

	生产者应用程序可以在任何地方运行，例如在服务器或设备上运行。

	生产的事件可以由生产者或中间人直接呈现和发布; 作为后者的示例，考虑由设备在诸如LoRaWAN或ModBus之类的有效载荷大小约束的网络上发送的事件数据，并且其中符合该规范的事件将由网络网关代表生产者呈现。

	例如，气象站通过LoRaWAN每5分钟发送一个12字节的专有事件有效载荷，指示天气状况。 然后使用LoRaWAN网关以CloudEvents格式将事件发布到Internet目标。LoRaWAN网关是事件生产者，代表气象站发布，并将适当地设置事件元数据以反映事件的来源。

2. 应用程序将事件用于显示，归档，分析，工作流程处理，监视条件和/或为业务解决方案及其基础构建块的操作提供透明度等目的。

	消费者应用程序可以在任何地方运行，例如在服务器或设备上运行。

	消费事件的应用程序通常会对以下内容感兴趣：

	- 区分事件，使得完全相同的事件不会被处理两次。
	- 识别和选择原始上下文或生产者指定的分类。
	- 识别事件相对于始发上下文和/或相对于挂钟的时间顺序。
	- 理解事件中携带的与上下文相关的详细信息。
	- 关联来自多个事件生成器的事件实例，并将它们发送到相同的使用者上下文。

	在某些情况下，消费应用程序可能对以下内容感兴趣：

	- 从原始上下文获取有关事件主题的更多详细信息，例如获取有关需要特别访问授权的已更改对象的详细信息。例如，HR解决方案可能出于隐私原因在事件中仅发布非常有限的信息，并且任何需要更多数据的事件消费者将必须用他们自己的授权上下文中从HR系统获得与事件相关的细节。
	- 在原始上下文中与事件的主题交互，例如在被告知刚刚创建了该blob之后读取存储blob。

	消费者的兴趣所在会激励生产者在事件中包含哪些信息。

3. 中间件将事件从生产者路由到消费者，或者转发到其他中间件。生成事件的应用程序可能会将因消费者需求而产生的某些任务委托给中间件：

    - 针对多个类别之一或原始事件上下文，管理许多并发感兴趣的消费者
    - 代表消费者根据事件的类别或原始上下文处理过滤条件。
    - 转码，如从JSON解码后编码为MsgPack
    - 改变事件结构的转换，例如从专有格式到CloudEvents的映射，同时保留事件的身份和语义完整性。
    - 即时“推送式”发送给感兴趣的消费者。
    - 存储最终交付的事件，由消费者的（“pull”），或者在延迟之后由中间件的（“push”）发起提取。
    - 监视事件内容或事件流以进行监视或诊断。

    为了满足这些需求，中间件将对以下内容感兴趣：

	- 可用于事件的分类或上下文化的元数据鉴别器，使得消费者可以表达对一个或多个这样的类或上下文的兴趣。例如，消费者可能对与文件存储帐户内的特定目录相关的所有事件感兴趣。
	- 元数据鉴别器，允许区分该类型或上下文的特定事件的主题。例如，消费者可能想要过滤掉与以“.jpg”结尾的新文件相关的所有事件（文件名是"新文件"事件的主题），用于描述它已经注册为感兴趣的文件存储帐户内的特定目录的上下文。
	- 用于编码事件及其数据的指示符。
	- 事件及其数据的结构布局（模式）的指示符。

	生产者的事件是否可通过中间件消费，是生产者的委托选择。

	在实践中，中间件可以在改变事件的语义含义时承担生成器的角色，在基于事件采取行动时承担消费者的角色，或在路由事件不进行语义更改时承担中间件的角色。

4. 框架和其他抽象使得与事件平台基础设施的交互更简单，并且通常为多个事件平台基础设施提供通用API区域。

	- 框架通常用于将事件转换为对象图，并将事件分派给某些特定的用户代码处理或用户规则，以允许使用应用程序对原始上下文和特定主题中的特定类型的事件作出反应。

	- 框架最感兴趣的是它们抽象的平台上的语义元数据共性，因此可以统一处理类似的活动。

	对于体育应用程序，使用该框架的开发人员可能对来自联盟中的球队（感兴趣的topic）的今天的游戏（subject）的所有事件感兴趣，但是想要处理与“substitution”的报告不同的“goal”的报告。为此，框架将需要一个合适的元数据鉴别器，使其不必理解事件细节。

## 符号和术语

### 符号约定

本文档中的关键词 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", 和 "OPTIONAL" 按照 [RFC 2119](https://tools.ietf.org/html/rfc2119) 中的描述进行解释。

### 属性命名约定

CloudEvents属性使用“camelCasing/驼峰法”作为对象成员名称，以有助于与常见的编程语言集成。

由多个单词组成的属性名称表示为复合词，第一个单词以小写字母开头，所有后续单词以大写字母开头，没有分隔符。

作为首字母缩写词的单词是全部大写的，例如 “ID”和“URL”。

### 术语

该规范定义了以下术语：

#### Occurrence/发生

"occurrence"是在软件系统的操作期间事实陈述的捕获。这可能是由于系统引发的信号或系统观察到的信号，因为状态改变，因为计时器耗尽或任何其他值得注意的活动。 例如，设备可能会进入警报状态，因为电池电量不足，或者虚拟机即将执行计划的重新启动。

#### Event/事件

"event"是表示发生(occurrence)及其上下文的数据记录。事件从发射源路由到感兴趣的各方，以便通知它们关于源的发生。可以基于事件中包含的信息来执行路由，但是事件将不标记特定的路由目的地。

#### Context/上下文

事件中包含关于发生的一组一致的元数据属性，工具和开发人员可以更好地处理事件。这些属性描述事件及其数据结构，包括有关原始系统的信息等。

#### Message/消息

事件通过消息从源传输到目的地。

#### Data/数据

关于发生/occurrence（如有效载荷）的领域特定信息。这可能包括有关发生的信息，有关已更改数据的详细信息或更多。

#### Protocol/协议

消息可以通过各种行业标准协议（例如HTTP，AMQP，MQTT，SMTP），开源协议（例如Kafka，NATS）或平台/供应商特定协议（AWS Kinesis，Azure Event Grid）来传递。

## 类型系统

以下抽象数据类型可用于属性。

- `String` - 可打印的Unicode字符序列。
- `Binary` - 字节序列。
- `Map` - 对象类型值的字符串索引字典
- `Object` - String`, 或 `Binary`, 或 `Map`
- `URI` - 符合 `URI-reference` 的字符串表达式，在 [RFC 3986 §4.1](https://tools.ietf.org/html/rfc3986#section-4.1) 中定义。
- `Timestamp` - 字符串表达式，在 [RFC 3339](https://tools.ietf.org/html/rfc3339) 中定义

此规范未定义数字或逻辑类型。

Object类型是一种变体类型，可以是String或Binary或Map。类型系统是有意抽象的，因此留给实现如何表示变体类型。

## Context属性

符合此规范的每个事件都必须包含上下文。

上下文被设计为可以与事件数据分开发送（例如，在协议header或协议特定属性中）。这允许在目的地检查上下文，而不必反序列化事件数据。对于某些用例，上下文可能还是需要对事件数据进行序列化（例如，JSON实现可能使用一个包含上下文和数据的JSON对象）。

### eventType

- 类型: `String`
- 描述：已发生的事件类型。此属性通常用于路由，可观察性，策略实施等。
- 约束:
  - 必须
  - 必须是非空字符串
  - 应该以反向DNS名称为前缀。前缀域名指示定义此事件类型语义的组织。
- 示例
  - com.github.pull.create

### eventTypeVersion

- 类型: `String`
- 描述：eventType的版本。 这使得最终消费者能够解释数据，要求消费者了解生产者。
- 约束:
  - 可选
  - 如果有，必须是非空字符串

### cloudEventsVersion

- 类型: `String`
- 描述：事件使用的CloudEvents规范的版本。这使得能够解释上下文。
- 约束:
  - 必须
  - 必须是非空字符串

### source

- 类型: `URI`
- 描述：描述事件生产者。通常包括诸如事件源的类型，发布事件的组织以及一些独特的标识符之类的信息。URI中编码数据背后的确切语法和语义是事件生产者定义的。
- 约束:
  - 必须

### eventID

- 类型: `String`
- 描述：事件的ID。 此字符串的语义未显式定义，以简化生产者的实现。启用重复数据删除。
- 示例:
  - 数据库 commit ID
- 约束:
  - 必须
  - 必须是非空字符串
  - 必须在生产者范围内唯一

### eventTime

- 类型: `Timestamp`
- 描述：事件发生的时间戳
- 约束:
  - 可选
  - 如果有, 必须符合 [RFC 3339](https://tools.ietf.org/html/rfc3339) 中定义的格式

### schemaURL

- 类型: `URI`
- 描述：指向`data`属性所遵循的模式的链接。
- 约束:
  - 可选
  - 如果有, 必须符合 [RFC 3986](https://tools.ietf.org/html/rfc3986) 中定义的格式

### contentType

- 类型: `String`
- 描述：描述数据编码格式
- 约束:
  - 可选
  - 如果有, 必须符合 [RFC 2046](https://tools.ietf.org/html/rfc2046) 中定义的格式
- 对于 Media Type 的示例，见 [IANA Media Types](http://www.iana.org/assignments/media-types/media-types.xhtml)

### extensions

- 类型: `Map`
- 描述：用于其他元数据的，并且没有强制结构。这为生产者或中间件可能想要包含的自定义字段提供了一个位置，并在将元数据添加到CloudEvents规范之前提供了测试元数据的位置。
- 约束:
  - 可选
  - 如果有，必须包含至少一个条目
- 示例:
  - 认证数据

### data

- 类型: `Object`
- 描述：事件负载。载荷取决于eventType，schemaURL和eventTypeVersion，载荷被编码成由contentType属性（例如application/json）指定的媒体格式。
- 约束:
  - 可选