---
title: "利用服务网格和智能应用感知网络增强应用弹性"
summary: "本文是作者在 Qcon 上的分享，主要谈及服务网格及其引申出来的应用感知网络。"
date: 2022-02-21T16:00:00+08:00
authors: ["Varun Talwar"]
translators: ["宋净超"]
categories: ["service mesh"]
tags: ["service mesh","istio","Tetrate"]
---

我是 Vrun Talwar，Tetrate 公司的联合创始人。我们是一家企业级服务网格公司。我要谈的是弹性，更准确地说，是运行时的弹性，是内置于你的网络中的东西。我喜欢从历史上的一个技术话题开始谈起。Cloud 1.0 是云的第一个时代。当时我们看到了虚拟化的浪潮，人们基本上从他们的硬件中获得更多。在我们进入当前的云时代之前，这已经持续了好几年，也就是 Cloud 2.0 时代，这基本上是从别人那里获得计算资源。你不需要在数据中心运行机器，别人为你更有效地运行它们。你刷一下信用卡，就可以得到他们管理的资源。这对配置灵活性和在我们想要的任何地方提供计算有很大的帮助。实际上，下一阶段就是 Cloud 3.0，这是一个更加动态和分布式的计算。从容器和自动伸缩的意义上讲，动态的，通过 Kubernetes 这样的协调器进行调度。分布式是指不同的区域：私有云、公有云、混合云等等。以及在应用组件分布的意义上的分布式。在一个计算如此动态的世界里，我们的网络和安全堆栈是滞后的。这些都是需要迎头赶上的。

## Cloud 3.0 转型 —— 网络的创新

在创办 Tetrate 之前，我曾有机会在谷歌工作了大约 11 年。很多人都在谈论，谷歌的基础设施怎么会如此可靠和安全？为什么它如此有弹性？尽管推出了更多的服务，尽管每年有数以千计的新开发者加入，但基础设施始终是正常的，可用的。这其中的核心之一是对核心网络的投资。谷歌的网络创新相当少，并不是所有的创新都被谈论过。我有幸参与了其中的两项重要创新，即 gRPC 和 Istio，我是这两个创新的共同创造者。这些都是网络栈被带到应用层面的地方。gRPC 是这个现代 RPC 结构，在 2016 年推出。Istio 是这种基于代理的方法，将代理注入到网络中，并使其成为 L7 代理，知道什么是通过它们进行的。这是在 2017 年推出的。这两个都是今天蓬勃发展的开源项目。

## 背景介绍

回到这次谈话的背景，弹性是超级重要的。随着越来越多的公司转向公有云，任何一个云供应商出现故障时，受到影响的品牌名单就会不断增加。这大大阻碍了他们的正常运行时间，不仅仅是正常运行时间，还有他们的业务和品牌形象。

## 弹性不仅仅与软件有关

我们如何才能做得更好？在我们讨论这个问题之前，让我们先来看看弹性问题的范围。这是一个多层次的问题，从基础设施层开始，然后延伸到网络层。它们的分布越多，网络层的可靠性就越关键。显然，也延伸到数据层，以及你的人、实践和操作。故障可能是不同类型的。你可以从一个主机到一个节点，到一个特定的服务，到一个特定的数据中心，到一个特定的区域。很明显，在物理层面上，在布线、交换机和路由器方面也是如此。所有这些都会给你的应用程序造成故障模式和可用性问题。问题是，你如何设计你的应用程序以适应它们？我们可以做得更好，而不仅仅是两种部署，主 - 主，或主 - 被？

在一个计算无处不在的世界里，我的观点是，你应该在多个可用区部署应用程序。无论如何，它们现在更容易配置、运行和管理。部署流水线更加自动化。我们真正需要的是一个智能的、连接的网络，它可以将流量一直路由到正确的健康部署，我们将有弹性的应用程序。说起来容易做起来难，我们如何才能在实践中真正做到这一点？让我们看看一些场景。

## 情景 1：服务实例失败

想象一下一个简单的三层应用程序的场景。你有你的前端 Web 服务器数据库，你有流量进入一些边缘。它可能是一个数据中心或一个云区域，进入一些应用代理或入口代理，然后进入你的应用程序。第一件事是应用程序应该部署在多个可用区。这是使其更具弹性的第一个前提。第二是模拟故障，并加强你的服务代码库处理故障的能力。像服务网格和 Istio 有一定的能力，你可以注入和模拟故障，并使它准备好更多的容错。一旦你在可用性区域部署了应用程序，你需要它有故障转移的东西是区域之间的连接，所以你实际上可以将流量路由过去。这些是提高可用性的一些良好做法。

## 服务代理 - 通往更健康的实例的路由

比方说，你有一个特定的服务实例在一个给定的节点上停机。它可能是数据库。它可能是网络服务器。它可能是前端。这个弹性网络的方法是在每个服务旁边有一个服务代理，或者在整个应用面前有一个应用代理，它可以检测到一个特定的实例正在发生错误。也许这可以通过更高的延迟或更高的错误率，或其他信号来检测，而且是来自该实例。它通常来自运行在它们旁边的 Sidecar 代理。这可以发出信号说，好吧，我应该把负载均衡到另一个更健康的实例，它有更健康的计算池，它有更健康的 pod，如果你遵循 Kubernetes。这是一个保持可用性和弹性的简单方法。另一个是关于，故障会发生。你如何确保代理足够聪明，内置超时和快速重试，所以他们可以从这些模式中恢复？这些也是很好的提示和做法。

## 情景 2：服务失败

假设整个服务瘫痪了，而且在那个特定的区域或特定的数据中心，没有一个实例是实际可用的。那你该怎么办呢？那么，你需要做的是将其路由到一个不同的可用区。这说起来容易，做起来难。要做到这一点，你需要知道每个服务和所有区域的状态和健康状况，并实时输入控制器，然后可以决定，好吧，我应该把流量发送到哪里？你需要它们之间的连接，以便它能够真正地路由流量。数据的一致性是另一个层面的问题，需要解决的是你要有一个一致的结果。另一个问题是，让这些东西在自动扩展的基础设施上运行总是可取的，所以资源容量不会成为我们可用性的一个问题。

## 可用性数学

有一件事我们都知道，但有时会忘记，用数字来表示是很好的，那就是，什么是可用性？可用性的定义是我在任何一年的平均停机时间是多少。我们经常谈论两个九、四个九、五个九的可用性，但实际上，只要在一到两个可用性区域内有可用性，就可以大大减少我们的停机时间，并提高我们的弹性。即使是一到两个可用区，也是非常有意义的影响。

## 情景 3：应用失败

继续这一趋势，让我们说，不是一个服务或服务实例，而是整个应用系统瘫痪了。那么，你如何将流量路由到该应用程序的一个完全不同的实例？如何设置两层的负载均衡，这样上面的一层，在这种情况下，边缘代理实际上可以知道，将流量发送到哪个应用代理。在这里，重要的是，你的所有其他安全控制，你已经建立的合规控制，你需要操作的应用程序，实际上在所有这些可用性区域都可用。这是通过服务网格的配置来完成的，这些 L7 网格，可以确保相同的配置被发送到所有的区域，因此你可以保证相同的行为。这看起来很容易，但对每个人来说，要实现这样的设置并不容易，即健康信号传播到边缘代理，代理做出正确的决定，你以正确的方式进行负载均衡。

## 情景 4：区域故障

你可以把这个问题提升到不仅仅是一个应用程序，而是整个区域的故障，整个数据中心的故障或容量不足。在这些情况下，你要把路由到一个完全不同的数据中心。在云设置中，它甚至可能意味着到一个完全不同的云。只要你有应用程序部署在这些区域，问题的解决方案是类似的，也就是你有一个上面的层，它在任何时候都有健康和性能的信号，并可以做出路由决策，将流量路由到最佳区域。然后从那里到最佳应用实例，再从那里到实际服务。

## 通过动态自动伸缩的 L7 网络实现复原能力

总之，我的主要观点是，我们可以有这种动态的自动伸缩，应用程序感知的网络。之所以称之为自动伸缩，是因为所有这些负载均衡器也可以在计算中运行，它可以自动伸缩，而且它们也可以是弹性的，就像你的计算节点。这种设置如果部署得当，架构合理，可以做两件事，第一，大大改善你的应用程序的弹性。二，你的开发人员不需要在他们的每个服务和应用程序中建立所有这些，并使其成为服务网格本身的一部分。我们在 Tetrate 是以这个为生的。我们有一个平台来实现这一点。在很多地方做了这些工作后，我们有不少最佳实践和蓝图架构，在实际的、真实的环境中很适合。

## 问答

**当你讲述谷歌的网络进化故事时，我想到的是，是什么导致了 gRPC 和 Istio 的诞生？你以前有什么不理想的地方，然后导致了最初 gRPC 的创建？什么问题没有得到解决？你也许想谈一谈这个问题。**

Talwar：gRPC 是谷歌内部一个叫 Stubby 的东西的下一代演绎。Stubby 在谷歌成立之初就存在了，也就是 1999 年。实际上，任何两个服务都可以通过这个 Stubby 机制相互通信。它存在了很长时间。在大约 12 年的时间里经历了一系列的迭代。那么需要它的原因有两个方面。在这种规模下，如果你在 HTTP 上做 JSON，这是客户端流量的经典方式，对于我们的规模来说，这还不够理想。只是给你一些例子，只是通过做 protobuf，也就是通过二进制，比通过文本，与通过 HTTP 的 JSON 相比，你在许多情况下得到 10 倍的改进。这意味着在我们的规模上可以节省数百万甚至数十亿美元的费用。

然后，渐渐地，发生了很多事情，比如负载均衡，重试，以及发送一些跨度进行追踪。gRPC 只不过是它的下一个版本，它被开放了源代码。原因是在一个组织中，你可以非常有主见，就像，好吧，我只支持三种语言或四种语言，在某些情况下，只支持一种语言，然后这些是我的库。这是好的。当我们把它放在外面的时候，你不能有那种一个组织的意见。谷歌实际上是用三种语言运行的。C++、Java 和 Python。然后一切都在这里面。当我们不得不进入多语言世界，并支持许多现有的服务时，这就是为什么需要一些不基于库和代理的东西，这导致了 Istio。

**有一种争论是，对于断路器来说，最好是避免回退或重试，而典型的情况是，它们需要在应用中实现，而不是在网络层。你有什么想法？**

Talwar：我们正在经历这样一个有趣的时代，什么是在应用程序中，什么是在网络中，在许多情况下，需要合作。与追踪不同的是，对于传递标头值，这是一个很好的例子，你必须在应用中做到这一点。断路器，核心代理，无论是 Envoy，还是其他代理，它们都有这些内置的概念，能够检查上游的健康状况，或者你要把它发送到哪里。定义我何时破坏它的规则，并通过配置做所有这些，所以这些范式存在于这些代理和控制平面中。显然，这一切只是基于代理，而不是基于通过代理的所有流量，就通过代理的请求的延迟和错误率而言。他们不知道你底层计算的其他方面。比方说，你的 CPU 超载了，就像那个应用程序正在消耗，这不会被知道。现在发生的更多的是这些事情被添加到了上面，也就是，从你的节点传递信号，比如 CPU 内存信号，这些被传递到了上面，以做出一些决定，或者能够从应用中获取外部信号，让代理做出决定。

显然，应用程序本身知道，有最多的背景，但人们在理解方面实际发展了多少，从节点开始，一直到可能出错的不同事情。我认为这很难。我们至少看到的两件事是代理与底层节点和应用程序之间的互动，以及反向的互动。这基本上意味着代理向底层自动扩展基础设施发出信号以进行扩展，所以这实际上也在更多地发生。我知道健康状况正在下降，因为延迟上升，信号下降到像 Kubernetes 这样的自动伸缩基础设施，或者只是云供应商的实例组。这是一个没有被使用的信号，应该被使用。

**有一种观点认为，Istio 没有被企业完全采用。你怎么看？企业需要认识到哪些事情，然后利用你所说的这种智能应用感知网络的优势？**

Talwar：Istio 在技术势头之前就已经有了营销的势头。这是其反馈循环的原因之一。它现在变得更好了。现在已经好了很多。另一件事是，它有太多的旋钮和太多的配置，等等，它只是让人们摸索和采用时变得复杂。另一件事是你需要对谁能做什么进行非常干净的控制。我经常告诉人们，与 Kubernetes 和其他类似的东西不同，**Istio 和服务网格总体上是一个多角色的问题**。这不是一个单一角色的问题。在企业内部，一个平台团队如何管理网关，管理 Sidecar？Sidecar 通常与应用结合在一起，所以现在，如何进行应用升级？这是应用团队的事。网关通常由一个不同的团队管理。如果你要一直走到边缘，通常有边缘代理团队。然后，安全希望总是作为其他人在那里，至少有可见性。在许多情况下，甚至想强制执行必须发生的策略，以及可选择发生的策略。他们甚至希望在工作流程中向外部暴露服务。

总之，你必须解决的问题是，每个团队在他们的观点和控制方面得到什么？你如何使旋钮更简单地使用？如果你问我，这里有太多的旋钮和太多的 YAML。有一件事是，只要让它简单，这是我的 API，这是我想要的行为。这应该就这样发生。像 Istio 这样的东西只是在平台上实现的，在基础设施中也是如此。这就是我们在 Tetrate 采取的方法。我认为这是一个长期的方法，如果这将被真正地大规模采用，而且时间更长。这就是它将成为的方式。像大多数技术一样，它将变得枯燥和不可见，将有一种方法可以直接使用它们，而不必对它们的细节进行搔扰。

**当我们谈论服务网格和 Istio，当然还有其他技术时，人们总是担心这与传统的 API 网关有什么关系。路线图变得模糊了。比如，什么是边缘代理？什么是 API 网关？它们有区别吗？它们应该是不同的吗？你对此有什么想法？**

Talwar：这里显然是有偏见的。我认为它们不应该是不同的。我们正在建立和构建的平台是这样的，全程使用 Envoy，因为你可以把它部署为一级负载均衡器，作为边缘代理。每个应用都可以有一个代理和一个应用代理在前面。然后，你可以拥有同样基于同一数据平面的 Sidecar。一个数据平面贯穿始终，然后每个应用程序，一个应用程序是我们的第一类概念，做它需要做的事情。在某些情况下，你会想，只要在入口层做认证，这就是我想要的一切。这就是我需要做的一切。我暂时不会去做 Sidecar 业务，这很正常。而其他人会想，不，我已经准备好了。这都是 HTTP，我很舒服。它不像超级性能敏感，延迟的东西对我来说并不重要。你也可以走这条路。

我思考的方式是，人们建立服务。人们部署服务。你可以通过内部 API 将它们暴露给你的内部团队成员，和 / 或你的合作伙伴。你可以通过公共 API 将其暴露给公众。你需要的控制是类似的。传统上的南北和东西之间的界限正在模糊化。人们做更多的微服务和 API 协议，你需要基于内部 API 的互动。唯一的区别是内部 API，你会在那里做基于令牌的认证。在外部 API 中，你会要求像，我需要有 OAuth，你需要通过这个流程。在外部 API 中，你要做的是，不，我想要 WAF 风格的策略，即批量保护这些 IP 的类型。而在内部 API 中，你将只是说，来自这个团队的测试流量不应该对我进行 DoS，所以只是速率限制。场景有一点不同，但技术层面的控制是相似的。我认为把它放在一个平台上是很有意义的，这就是唯一的区别。

事实上，Istio 所来自的团队，实际上被称为 One Platform，这是谷歌的说法，这是内部 API，外部 API。你只要告诉我们你在你的 API 中想要什么。这些是行为。我们以前在谷歌的团队所做的是，每个团队只是提交他们的 API 规格和他们想要的东西，事情就会发生。今天它是一个内部的，明天就变成了一个外部的 API。你可以在 API 规格上添加一些东西，仅此而已。在推广方面没有其他变化。

**网格的网格，这到底是不是同一个东西？**

Talwar：我不太喜欢这个词，但这个概念确实是真的。我们在 Tetrate 所做的，以及我认为更普遍的，在工业界，它正在成为事实，这就是，有三个层次，这还没有被很好地解释。有一个数据平面，它必须通信的地方。还有一个控制平面，需要在它的附近，也就是在同一个集群或同一个 VPC 中，但不能太远。然后还有第三层，也就是我们所说的管理平面，也就是你在上面看，说，好的，我需要为每个应用做什么，做路由决定，做弹性决定，等等。我们正在建立管理平面。Istio 仍然是按原样使用的，随着它的功能不断增加，以及所有这些，作为附近的控制平面。

当然，它是以一种与计算和云无关的方式完成的。如果我在微软云有 N 个集群，在亚马逊云有 N 个集群，每个集群都可以用 Istio 作为控制平面。你真的能做出那些有弹性的决定，而不是路由到这个微软区，而是路由到这个亚马逊区？人们来问我，为了成本、性能、安全或其他原因选择他们喜欢的云服务，我们可以这样做吗？这绝对是可能的，但对人们来说并不那么容易实现。我们想让这个目标容易实现。我认为我们正在走向那个世界。网格的网格听起来是一个不好的名字，但从架构上看，我们正在走向那里。

是的，它可以在任何地方。这就是管理机的魅力所在。它可以在任何一个地方，无论你决定它在哪里。问题是，所需的边缘或入口要放在更接近其应用的地方。

**我喜欢你的智能应用感知网络的说法，你认为它是否总是可以自动等同于一般的服务网格？如果我不使用边缘技术，那么我的替代方案是什么，如果有的话？**

Talwar：服务网格成为这个一切的术语。这个概念就是你的网络和你的平台层更加智能。例如，gRPC 是我的另一个宝贝，它支持 xDS，所以你可以在 gRPC 中建立东西，没有任何代理，并要求控制层有同样的行为，这一切都可以。我认为更多的语言栈和框架将开始支持这些类似 xDS 的功能。一旦他们这样做了，我想我们就可以进入事情本身的构建方式了。这还没有发生。我认为这才是应该发生的。今天是 Envoy 代理。明天可能是更好的东西。这个概念在语言框架和这些代理中都是一样的。让它们更智能，这样你就不会在应用中做了，而且你可以在不加重应用开发者负担的情况下，在你的应用中持续地做。我认为这个概念是非常有用的，而且会继续存在。