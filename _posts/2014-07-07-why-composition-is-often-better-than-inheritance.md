---
layout: post
title: "[译]为什么组合经常比继承更合适"
date: "2014-07-07 19:55"
description: ""
category: 
tags: [composition, inheritance]
---

译自 [Why composition is often better than inheritance](http://joostdevblog.blogspot.com/2014/07/why-composition-is-often-better-than.html)

代码结构中一个重要的问题是应该使用组合还是继承来联系不同的类，即类的关系应该是"has a" 还是 "is a".
举个例子，一把椅子有一个坐垫 和 一把椅子是一件家具。此例中的不同是显而易见的，但在实际中，许多时候就不
那么容易区分。比如，游戏中的一个角色应该有一个包围盒，还是本身就是一个可碰撞的实体？是不同的，虽然都可以作为碰撞
处理的主要结构（或者两种结合起来）, 并且很难说那种更好。我的经验直觉经常倾向于继承，但是这样会有很多
问题，以至于很多时候组合会更好一些。

让我们先看两种方式都适用的一个例子。在[Awesomenauts][]中，我们有一个独立的类
来处理角色的物理特性，比如重力、反弹、滑动、以及跳跃。可以一个角色就是一个物理实体，这样将会是Character继承自
PhysicsObject。同样可以说一个角色拥有一个PhysicxObject来处理物理碰撞，此时经常会使用 "uses a" 替代 "has a".

![](http://www.proun-game.com/Oogst3D/BLOG/Inheritance%20versus%20composition%20-%20Basic%20scheme.gif)

继续看在代码中是如何表达这种关系的。肃然是一个比较简单的版本，但已经足够说明基础概念：

``` cpp
class PhysicsObject
{
 Vector2D position;

 void updatePhysics();
 void applyKnockback(Vector2D force);
 Vector2D getPosition() const;
};

//Using PhysicsObject through inheritance
class CharacterInheritance: PhysicsObject
{
 void update()
 {
  updatePhysics();
 }
};

//Using PhysicsObject through composition
class CharacterComposition
{
 PhysicsObject physicsObject;
 
 void update()
 {
  physicsObject.updatePhysics();
 }
 
 void applyKnockback(Vector2D force)
 {
  physicsObject.applyKnockback(force);
 }
 
 void getPosition() const
 {
  return physicsObject.getPosition();
 }
};
```

如代码所示，Character使用继承更简洁，也感觉更加自然，因为我们不用再为KnockBack和getPosition写访问方法。然而，经过
多年的对同样问题的的处理，我学习到，使用组合比使用继承，更加灵活，更少的bug，设置更容易理解。


灵活性(Flexibility)
-------------------

首先说灵活，假设我们需要实现一个敌人，它由两个块以及连接它们的能量链组成. 触碰链条则会被伤害。两个块可以独立移动，
在于敌人打斗时可以使其处于有趣的位置。每个块都有其独立的物理特性. 抨击其中一个块，不会影响到另外一个。但是它们确实
是一个角色：有着同样的生命条、一个AI，小地图中的一个点，以及永远只能同时存在。

![](http://www.proun-game.com/Oogst3D/BLOG/Inheritance%20versus%20composition%20-%20Enemy.gif)

使用组合来创建这个敌人，将会相当直观，我们可以直接创建一个拥有多个PhysicObject的角色。若使用继承，我们无法用一个角色创建
，因为无法继承PhysicsObject两次。当然我们使用继承终究可以实现，但是将不会简单和直观。

如果你觉得这个例子比较牵强，那可能是因为你还没有参与一个大型的游戏项目。游戏策划不断的提出超乎你已实现的游戏逻辑意外的需求。
仅仅应为你的代码结构不能处理而拒绝这些需求将会损害游戏质量，因为大家的目标应该都是这个游戏是否好玩(好吧，先不管这个项目
更不能实现这个目标).

只要看一下[Awesomenauts]中数百的各种各样的升级, 你就能明白有多少是我们的当时的代码无法实现的。我们的策划提出这些需求，我们
的代码就要保证它能实现。游戏编程中最重要的目标就是灵活：使你的代码能够方便的添加今天策划突发奇想的需求。很多时候，组合比
继承更加灵活。

可读性(Readability)
--------------------

我下一个反对继承的点是可读性。可读性往往会伴随这bug的多少，如果一个程序员无法真正理解他所工作的事物如何运行，那他将很有可能破坏它.

在[Ronimo][], 我们编码准则中最重要的一条是我们力争保持所有的类少于500行代码。我们并没有一直去检查保证，但是目标是很清晰的，那就是：
保持类相当的简短以便其容易理解，这样程序员在脑海中就能处理这个类所有的工作。

上面提到伴随着游戏不断增加特性，我们的代码不断增长, Character 和 PhysicsObject 这两个类也达到了500行。同样我们增加了两个新的类使用
PhysicsObject：PickUp 和 Projectile，同样达到500行。

![](http://www.proun-game.com/Oogst3D/BLOG/Inheritance%20versus%20composition%20-%20Pickup%20Projectile.gif)

这种情况下继承会是问题非常困惑。我们尽可能的保持类公开的数据和接口更少，但是继承最后会引入许多virtual 和 protected 方法。在这个例子
中，PhysicsObject 将会和他的三个子类：Character、PickUp和Projectile之间相互调用。我的经验中经常会发生这样的事情：随着时间，继承的子类需要一起工作
以便产生复杂的行为和相互调用。

如果没有真正的理解这些类，将不能更好的明白这个问题，现在让我们来继续看。如果修改PhysicsObject以满足Projectile的一个新特性，那么Character
和PickUp也会收到影响。为了了解整个情况，程序员现在必须同时考虑四个不同的类, 这将接近2000行代码。我的经验是这是不可能的：因为有太多的代码
去理解，这还不算其中又混入其他代码。结果是可读性降低，而程序员也更容易犯错，应为他很容易忽视一些事情。

当然基于结构的组合并不是总能神奇地解决所有这些问题, 但它确实会使结构变得简单和容易理解。使用组合将不会有virtual 和 protected，因此PhysicsObjec
和Charater、Projectile、PickUp 是完全独立地. 这样保证类不会随着时间交织在一起，而且更加容易做到真的独立。使用继承，很难像组合强制保持两个理论上
独立地类真正地互不影响，我们代码中有大量地例子，随着代码增多，继承地类很难去理解。

钻石问题(Diamond problem)
-------------------------
(译注：菱形继承问题)

任何关于继承和组合地讨论，都会提到著名地钻石问题。类A继承自两个类B和C， 而B和C又都继承自同一个类D，这时会发生什么？A现在有两个D，很困惑。

![](http://www.proun-game.com/Oogst3D/BLOG/Inheritance%20versus%20composition%20-%20Diamond.gif)

对于钻石问题，C++有多种解决方案。你可以接受A有两个D(haha, double D! Ahum). 或者你可以使用虚继承。然而，每种办法都会产生许多问题和潜在地bug,
因此避免钻石问题才是首选地方案。

这个问题一般不会出现在游戏代码结构设计地初始阶段，但是随着特性地增加，很有可能发生。而且一旦产生，除了重构，经常很难有好的解决办法.

尽管如此，钻石问题只出现了两次在我12年地面向对象编程中，因此我不觉得钻石问题是反对继承的强有力的理由。

使用继承，但是少用(Use inheritance, but use it less)
----------------------------------------------------

说到这里，是不是意味着我反对继承? 不，当然不是！很多时候继承是非常有用的，比如多态(polymorphism)以及观察者、工厂方法等设计模式。我的观点是
继承并不像它看起来那么好用。许多例子中，继承看起来是一个简洁的设计，但是实践中发现组合更加适合。所以使用继承，但是不要滥用。


[Awesomenauts]:http://www.awesomenauts.com/
[Ronimo]:http://www.ronimo-games.com/
