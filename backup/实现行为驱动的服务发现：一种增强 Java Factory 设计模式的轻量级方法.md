[Factory 模式](https://dzone.com/articles/java-the-factory-pattern)是 [Java](https://dzone.com/refcardz/core-java) 中流行的创建设计模式之一。它提供了一个访问点，通过使用服务标识符（通常是实现类型的简写名称，由 String 或 Enum 表示）来获取作为抽象类或接口公开的服务的合适实现。此模式用于创建对象，而无需指定将在运行时创建的确切对象类，开发人员每天遇到的许多框架和 Java API 都使用这种模式。

本文旨在增强设计模式以提供更好的可读性和可维护性。它说明了如何通过建议的方法自动进行[服务发现](https://dzone.com/articles/microservices-architectures-what-is-service-discov)，而无需工厂方法在每次为不同的业务需求创建新的服务实现时手动容纳用于创建新服务实现的代码。

本文的受众假定之前已经接触过 Java 编程语言以及设计模式的基本概念。但是，我们将在下一节中介绍 Factory Design Pattern 的基础知识，以便我们可以详细说明和展示我们提议的增强如何在我们将在那里介绍的示例中工作。

## 1\. 工厂设计模式：快速修订

Factory Design Pattern 是一种创建模式，它提供了一个在超类中创建对象的接口，但允许子类更改将要创建的对象的类型。当在运行时确定要创建的对象的确切类型时，此模式特别有用。

###  关键组件

1.  **服务：** 作为接口公开的抽象业务功能
2.  **服务实施：** 这些是上述接口的实现类。当然，它们实现了为不同但相似的业务功能定义的不同抽象方法。
3.  **工厂方法：** 这通常是 creator，通常简化为类中的静态方法，称为`工厂`类，它根据通过消费者代码传递给它的参数返回适当服务实现类的对象。

###  示例场景

仓库根据物料的材质对物料进行包装。他们非常小心地防止寄送的物品在运输途中损坏。例如，包装易碎物品必须与包装需要防止极端温度和冲击的电子产品不同。他们为每类物品都有专门的包装机。因此，我们可以假设，如果 `Packer` 是一个抽象类型，那么找到合适的 Packer 就是需要了解要打包的物品类型的人。代码片段快速描述了场景：

#### 包含代码片段的示例

1.  我们有一个 POJO 代表仓库中的物品：  
    ![POJO representing the items in the warehouse](https://dz2cdn1.dzone.com/storage/temp/18097397-1733940915583.png)
2.  服务 `Packer` 作为接口公开：![The service Packer is exposed as an interface](https://dz2cdn1.dzone.com/storage/temp/18072823-1732967094853.png)
3.  我们为接口提供了这些耦合的实现：![Coupled implementations for the interfac](https://dz2cdn1.dzone.com/storage/temp/18072824-1732967135850.png)
4.  最后，类 `PackerFactory` 有一个方法，可以根据传递给它的参数返回合适实现的对象。![The class PackerFactory has a method to return an object of the suitable implementation based on the argument passed to it](https://dz2cdn1.dzone.com/storage/temp/18072826-1732967167646.png)

一个简单的消费者代码如下所示：

![A close-up of a computer screen](https://dz2cdn1.dzone.com/storage/temp/18097403-1733941247016.png)

## 2\. 问题陈述及其背后的动机

让我们假设仓库要容纳更多类别的物品，如药品、易燃、不易燃液体等。对代码库的更改不仅包括 `Packer` 的相应实现，还包括工厂方法 `getSuitablePacker（String）` 中需要的更改，以适应这些实现。

![Implementations of Packer](https://dz2cdn1.dzone.com/storage/temp/18072828-1732967328223.png)

___

![Change needed in the factory method getSuitablePacker(String) to accommodate those implementations](https://dz2cdn1.dzone.com/storage/temp/18097424-1733942925268.png)

这些实现可能彼此独立，不需要或只需要服务接口来实现。工厂方法必须了解每个服务实现，并且业务条件必须基于方法参数返回适当的服务实现，即上例中的 `itemCategory`。新的实施可能不会一次性推出，但通常会在单个发布周期内推出一个或几个。

我们的目标是将开发人员从编写和维护工厂方法中解放出来。通常，不同的小团队负责服务的不同实现并维护工厂方法。尽管有版本控制系统，但团队在尝试适应各自的服务实现（或服务实现的版本：通常相同的服务实现的几个或多个版本共存;直到较旧的版本顺利退出，但这超出了我们讨论的范围）时，总是有机会——无论多么小——团队可能会相互覆盖。 DevOps 等_。_ 

还有另一个用例：通常，服务实现可以作为单独的工件发布。工厂方法本身可能是一个，为什么我们仍然应该认为它对开发开放，只是为了添加一些样板代码以适应更新的服务呢？我们强烈认为我们不应该在工厂方法本身中编写任何样板代码。事实上，如果我们永远不必编写和维护一个，那就更好了，至少对于一个简单的用例来说！

##  3\. 建议的方法

给定一个定义的服务抽象（接口），我们的目标是解决以下问题：

1.  使用相应的查找标识符标记服务实现（如上例中的 `itemCategory`）
2.  卸载工厂方法的开发和后续的样板维护

###  3.1 工艺流程

此过程流程说明了如何实现它：

![Process flow](https://dz2cdn1.dzone.com/storage/temp/18072830-1732967477069.png)

我们将详细讨论每个步骤，然后提供支持代码片段。值得注意的是，相同的流程可以通过多种方式实现，具体取决于所使用的框架（例如 [Spring](https://dzone.com/articles/spring-framework-tutorial-for-beginners-2)）和[依赖项注入](https://dzone.com/articles/dependency-injection-in-spring)容器。因此，我们强烈建议您以最适合您的项目环境的方式实现目标，因为我们将保持与任何仅遵循 Vanilla Java 8 的框架无关。

首先，我们必须介绍以下术语，这些术语将在本节及以后经常提到。

####  **Service Registry 映射**

这是一个跟踪`服务`接口实现的映射。我们使用嵌套的 `java.util.HashMap` 实现来表示此数据结构。

外部 `Map` 的键是`服务`接口的 `java.lang.Class`，该值本身是一个 `Map`，其键是**绑定标识符**（在下一节中讨论 - 现在可以跳过这个术语），其值是服务实现对象。我们从第 1 节开始就一直在讨论的 `Packer` 接口的示例服务注册表映射如下所示：

![Sample service registry map for the Packer interface](https://dz2cdn1.dzone.com/storage/temp/18072831-1732967556501.png) 

`键`是 `Class<Packer Interface>`，`值`是一个映射，其中包含 `GlassPacker` 和 `ElectronicsPacker` 实现的对象，分别针对键“glass”和“electronics”。

#### **服务绑定标识符**

尽管一开始对你们中的许多人来说这似乎很吸引人，但可能您现在已经意识到它与 `Packer` 示例中的 `itemCategory` 相同。这确实有助于工厂方法选择服务接口的正确实现。它通常是人类可读的标识符，例如 String 或 Enum。

> **_使用 Enum 而不是 String 是一个好主意，尽管为了简单起见，我们在示例中使用了 String。_**

#### **填充服务实现注册表**

这是一个在后台动态加载 **Service Registry Map** 中的元素的过程**。** 我们将在 Section 3.2 中详细讨论这一点。

####  **服务元信息**

Service Meta 信息告诉我们 service interface 和相应的绑定。值得一提的是：请注意复数形式 _bindings_。我们遇到过这样的情况：单个服务实现适用于多个场景。例如，`GlassPacker` 可以满足与 `BrittleGlassUtensilsPacker` 之类的东西完全相同的需求。因此，前者也可以与 `itemCategory =” brittleGlassUtensils”` 一起使用;`GlassPacker` 适用于`“玻璃”`和`“脆性器皿”。`

> _简而言之，我们预置了 1 对多基数。_

####  **提供的 Factory Method**

这是一段预先编写的代码，它使用绑定从 **Service Registry Map** 中获取正确的实现。下面的代码片段显示了它的实现。但是，我们稍后也会回到这一点。             

![Implementation from the Service Registry Map, using bindings](https://dz2cdn1.dzone.com/storage/temp/18072833-1732967720261.png)

> _请注意，上述方法只是为了简洁地说明文章。它有很大的改进空间。_

### 3.2 填充服务实现注册表：一种简单的机制

到目前为止，我们已经看到，服务注册中心填充了服务元信息，并使用字符串标识符来获取合适的服务实现。这可以通过多种方式进行设计，具体取决于框架，尤其是你使用的用于依赖项注入的框架。但同样，我们只遵守 Vanilla Java。

![Populating Service Implementation Registry](https://dz2cdn1.dzone.com/storage/temp/18072834-1732967982412.png)

> _当您使用 Spring、Micronaut、Dagger、Guice 等框架时，这种方法的适应更加容易。其中一些方法与我们如何使用 Java 实现最低限度工作的原型大致一致。_

我们通过 annotation 来定义服务元信息。我们相应地对服务实现类进行注释。

我们有两个属性：一个是目标服务接口的 `java.lang.Class`，另一个是 `String` 数组，包含定义服务实现的绑定。

![java.lang.Class and String array](https://dz2cdn1.dzone.com/storage/temp/18072837-1732968069505.png)

`GlassPicker` 服务实现类如下所示：`绑定`包含查找键，并且工厂方法将使用`“glass”`来查找合适的 `Packer` 实现（如果可用）。这里值得一提的是，属性 `targetService` 似乎是多余的。为什么开发人员必须关心重新声明类实现的内容？我们在工作开始时也有同样的感觉，并且没有它就继续前进。然而，在我们设想的实际项目中遇到了一些复杂的遗留类型层次结构的障碍后，我们适应了这一点。

> _您必须有动力摆脱此属性，除非您的项目涉及复杂的多级层次结构！_

正如我们在本文开头提到的，我们意识到服务实现可能会在不同的工件中分阶段推出。我们利用 Java SPI（Service Provider Interface）来查找服务植入。这有助于缩短应用程序启动时间。实现类的递归检查成本太高了！

对于普通用例，该逻辑很简单：

1.  使用 Java Service Provider Interface 收集服务实现类（您可能喜欢[这个简短的复习](https://www.baeldung.com/java-spi)）。
2.  解析 implementation classes 上的 annotation 以获取 `targetService` 和 `bindings`。

填充服务注册表映射，保留 `Class<Service Interface>` 作为外部映射的`键`，并保留 binding `的值`作为嵌套映射的键。服务实现类的对象是内部映射的值。

执行这一小段验证是为了解决在注释服务时出现的人为错误。 

![This small piece of validation is performed to account for any human error while annotating the service](https://dz2cdn1.dzone.com/storage/temp/18072843-1732968293075.png)

 实现类：

![Implementation classes](https://dz2cdn1.dzone.com/storage/temp/18097422-1733942831049.png)

### 3.3 在简单的 Java 应用程序中访问 Service Registry

尽管应用程序永远不会像 `“Hello， World！”`Java `类，它们`无处不在，以方便学习任何内容，在知识转移期间，在技术或框架的自我评估期间。因此，我们尽可能简单地呈现这部分。观众必须根据自己的需要使用它。

请注意，为简单起见，我们只考虑了一个服务接口。方法 `lookUpService（Class<T> cls）` 可以对同一方法的重载版本中的任意数量的类调用。

![The method lookUpService(Class<T> cls) maybe called on any number of classes from an overloaded version of the same method](https://dz2cdn1.dzone.com/storage/temp/18072845-1732968497912.png)

### 3.4 关于打嗝的重要说明

在将这种机制部署到项目环境中时，我们面临的最不可避免的问题是 Java SPI 的可用性。我们有 100 多个 implementation 类，其中有一些服务已经存在多年了。幸运的是，它们要么归我们所有，要么没有关闭进行维护，我们可以继续进行相关更改，例如在 **src/main/resources/META-INF/services** 文件夹下注释类和声明服务实现（这是我们为 Maven 项目声明服务的地方）。如果我们没有求助于专门编写的 Script 来执行此重构，这将是一个非常乏味、令人沮丧且容易出错的重构 — 但这是一个完全不同的故事，超出了我们当前的范围。但是，如果您的项目环境中的服务接口已关闭进行开发，或者由于许可证问题或合规性限制而无法合法部署您所做的更改，那么 Java SPI 和注解方法都不适合您，至少对于现有服务是这样。但是，较新的服务可能仍会从此方法或类似方法中受益。

1.  在 Java SE 1.8 之前，我们通过在 中 `META-INF/services/ fullyQualifiedServiceInterface` 声明其完全限定的实现类名来公开服务。需要在文件中 `META-INF/services/com.somecompany.fooService` 声明已实现的服务 `com.anothercompany.pkg1.SomeFooServiceImpl` `com.somecompany.fooService`，其内容将为 `com.anothercompany.pkg1.SomeFooServiceImpl` 。但是，从 Java 9 开始，引入了模块。尽管大多数升级到更高和 LTS 版本的 Java 的遗留项目很少采用模块化，但如果您必须使用模块化，则需要相应地公开服务（老实说，这更容易！ _![Expose a service by declaring its fully qualified implementation class name in META-INF/services/ fullyQualifiedServiceInterface](https://dz2cdn1.dzone.com/storage/temp/18072848-1732968582733.png)_ 
2.  值得一提的是，这种方法不一定鼓励您一次加载 service registry 中的所有服务。但是，在此处预置了在需要时加载它们的功能。这不会影响现有的 service registry。
3.  这种方法是合适的，并且可以有效地扩展以实现**抽象工厂模式**。

通过 SPI 加载服务存在 `ServiceLoader` 的常见缺点。除非我们正在做一些事情来自定义 class loader 和 listeners 以实现动态性，否则这应该不会打扰你。

### 3.5 尚未讲述的内容

我们没有告诉你，我们的项目环境包括 Spring 框架和 [Spring Boot](https://dzone.com/articles/spring-boot-nice-amp-easy-video-8)。我们使用了注释服务实现的概念，但使用 Spring 容器很容易实现服务注册表映射。在 Spring Boot 中，您可以通过以下方式获取接口的实现：

-   服务实现类应标记为 `@Service` 或 `@Component`（如果适用）。
-   在 `FactoryService` 类（本身是一个`@Service`）中，可以注入 `List<ServiceInterface>` 类型的字段。 请注意，至少存在一个实现，以防止应用程序由于依赖项不满足而无法启动。否则，您可以将依赖项标记为非强制性或使用注入字段的 `java.util.Optional`。您可以选择最适合您的构造函数与字段注入，因为它非常基于意见。但我们_更喜欢_ 构造函数注入。此外，某些服务类可能很重，您可以决定是否希望延迟加载它们;在这种情况下，需要采取适当的措施，确保我们只在加载了所有 bean 时才填充 Service Map。其余的东西，比如解析 annotation 等，与我们在 **3.2** 节中演示的方法相同。

##  最后的想法

最后，我们总结说编译时依赖注入要快得多，我们的目标是专门为此目的写一篇文章。



非原创翻译自：https://dzone.com/articles/enabling-behaviour-driven-service-discovery-a-ligh
