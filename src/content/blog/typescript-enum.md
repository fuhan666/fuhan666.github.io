---
title: TypeScript 枚举的多种实现方式
publishDate: 2025-04-10 10:22:35
description: 'TypeScript 中实现枚举的多种方式，包括传统枚举、const 对象、字面量联合类型、Unique Symbol 和品牌化类型，分析各种方式的优缺点和适用场景。'
tags:
  - TypeScript
  - 枚举
language: 'Chinese'
---


## 使用传统 TypeScript 枚举

传统枚举是 TS 内置的语法糖，写法简洁，同时既可以作为类型又可以作为值使用。但缺点在于编译时会生成额外的代码，不适用于纯类型擦除的环境。

```typescript
// 定义一个传统枚举，表示方向
enum Direction {
  Up,    // 默认值为 0
  Down,  // 默认值为 1
  Left,  // 默认值为 2
  Right  // 默认值为 3
}

/**
 * 根据传入的方向进行处理
 * @param direction - 枚举类型 Direction 的一个成员
 */
function move(direction: Direction) {
  // 根据枚举变量进行分支判断
  switch (direction) {
    case Direction.Up:
      console.log("Moving up");
      break;
    case Direction.Down:
      console.log("Moving down");
      break;
    case Direction.Left:
      console.log("Moving left");
      break;
    case Direction.Right:
      console.log("Moving right");
      break;
  }
}

// 使用时直接利用枚举对象访问成员
move(Direction.Left);

// 优点：语法简单、直观，TypeScript 能同时利用类型和值支持。
// 缺点：会生成额外的运行时代码，在仅做类型擦除的场景下可能不适用。
```

---

## 利用 const 对象与 `as const`​ 实现枚举替代方案

这种方式借助对象字面量以及 `as const`​ 断言，保证生成的 JS 代码和手写代码一致，同时利用 TypeScript 提供的字面量类型推导生成一个联合类型。

```typescript
// 定义一个包含各个方向的对象，并通过 as const 让每个属性都是字面量类型
const DirectionObj = {
  Up: "up",
  Down: "down",
  Left: "left",
  Right: "right",
} as const;

// 利用 ValueOf 辅助类型（取对象值的联合类型），得到 "up" | "down" | "left" | "right" 的联合类型
type DirectionObj = (typeof DirectionObj)[keyof typeof DirectionObj];

/**
 * 根据传入的方向（联合类型）进行处理
 * @param direction - 只能是 "up"、"down"、"left" 或 "right"
 */
function move(direction: DirectionObj) {
  if (direction === DirectionObj.Up) {
    console.log("Moving up");
  } else if (direction === DirectionObj.Down) {
    console.log("Moving down");
  } else if (direction === DirectionObj.Left) {
    console.log("Moving left");
  } else if (direction === DirectionObj.Right) {
    console.log("Moving right");
  } else {
    // 理论上永远不会到这里，因为 direction 类型被严格限定
    console.error("Invalid direction");
  }
}

// 直接使用对象的成员调用函数
move(DirectionObj.Right);

// 优点：生成的代码与书写的对象字面量一致，没有额外代码，适用于代码体积敏感的环境；同时可组合、迭代、与 JS 操作无缝结合。
// 缺点：声明稍长，需要通过辅助类型 ValueOf 来提取联合类型，对比枚举语法来说稍显冗长。
```

让我们分步解释：

1. ​**​`const DirectionObj = { ... } as const`​**​

    * ​`const DirectionObj = { ... }`​: 这声明了一个名为 `DirectionObj`​ 的常量（`const`​）变量，并将其初始化为一个对象字面量（`{ ... }`​）。这里的 `{ ... }`​ 代表该对象包含一些键值对，例如：

      ```typescript
      const DirectionObj = {
        Up: "up",
        Down: "down",
        Left: "left",
        Right: "right",
      }
      ```
    * ​`as const`​: 这是 TypeScript 中的 **const 断言 (const assertion)** 。它告诉 TypeScript：

      * 推断出最具体的类型，而不是通用的类型。例如，`'up'`​ 的类型被推断为字面量类型 `'up'`​，而不是 `string`​。
      * 将所有属性标记为 `readonly`​。
      * 对于数组，将其视为 `readonly`​ 元组。
      * 在这个例子中，应用 `as const`​ 后，`DirectionObj`​ 常量的类型会被推断为：

        ```typescript
        // (假设使用了上面的例子)
        // typeof DirectionObj 的类型是:
        Readonly<{
            Up: "up";
            Down: "down";
            Left: "left";
        	Right: "right";
        }>
        ```
2. ​**​`type DirectionObj = (typeof Status)[keyof typeof DirectionObj]`​** ​

    * ​`type DirectionObj = ...`​: 这声明了一个名为 `DirectionObj`​ 的**类型别名 (Type Alias)** 。注意，这里的 `DirectionObj`​ 是一个 *类型*，它与上面定义的 `DirectionObj`​ *常量* 不同（TypeScript 允许值和类型共享名称，因为它们存在于不同的命名空间）。
    * ​`typeof DirectionObj`​: `typeof`​ 操作符在这里用于获取 `DirectionObj`​ *常量* 的类型。正如上面推断的，它的类型是 `Readonly<{ Up: "up"; Down: "down"; Left: "left"; Right: "right"; }>`​。
    * ​`keyof typeof DirectionObj`​: `keyof`​ 操作符用于获取一个类型的所有公共属性名的**联合类型**。对于 `typeof DirectionObj`​，`keyof`​ 会得到 `'Up' | 'Down' | 'Left' | 'Right'`​。
    * ​`(typeof DirectionObj)[keyof typeof DirectionObj]`​: 这是**索引访问类型 (Indexed Access Types)**  或 **查找类型 (Lookup Types)** 。它会查找 `typeof DirectionObj`​ 类型中，由 `keyof typeof DirectionObj`​（即 `'Up' | 'Down' | 'Left' | 'Right'`​）指定的那些键所对应的 *值* 的类型。

      * ​`DirectionObj['Up']`​ 的类型是 `'up'`​
      * ​`DirectionObj['Down']`​ 的类型是 `'down'`​
      * ​`DirectionObj['Left']`​ 的类型是 `'left'`​
      * ​`DirectionObj['Right']`​ 的类型是 `'right'`​
      * 因此，`(typeof DirectionObj)[keyof typeof DirectionObj]`​ 的结果是这些值类型的**联合类型**：`'up' | 'down' | 'left' | 'right'`​。
    * 对于这行代码，还有另一种写法

      ```typescript
      // 辅助类型：将对象 T 的所有属性值联合起来
      type ValueOf<T> = T[keyof T];
      type DirectionObj = ValueOf<typeof DirectionObj>;
      ```

      这两种写法在类型层面上是完全等效的。很多时候，我们会定义一个辅助类型别名（常叫做 `ValueOf<T>`​）来提取一个对象所有属性值构成的联合类型。它取得了 `DirectionObj`​ 的所有键（`keyof typeof DirectionObj`​）对应的值，并生成一个联合类型。因此，本质上它们没有区别，只是一个是把操作封装成了一个通用的别名，以便在多个地方重用，而这种写法在这个例子中则是内联展开，避免了额外的类型别名声明。

      这种选择主要是代码风格和复用性上的考虑。如果项目中有多个场景需要提取对象值的类型，使用 `ValueOf`​ 辅助类型会让代码更简洁和具有一致性；而在示例中直接内联写出相同表达式也能达到相同的效果。

**总结:**

这段代码的目的是：

1. 定义一个包含一组相关常量值（通常是字符串或数字）的对象 `DirectionObj`​。`as const`​ 确保这些值被推断为精确的字面量类型并且对象是只读的。
2. 基于这个常量对象的 *值*，自动创建一个**联合类型** `DirectionObj`​。这个联合类型包含了对象中所有值的字面量类型。

---

## 使用字面量联合类型

如果仅需要一个离散的值集合，而不关心既能作为类型也能作为值的语法糖，直接使用联合类型也是一种简单方案。

```ts
// 直接定义一个字符串联合类型，表示方向
type Direction = "up" | "down" | "left" | "right";

/**
 * 根据传入的方向进行响应处理，switch 结构实现分支判断
 * @param direction - 必须是 "up"、"down"、"left" 或 "right"
 */
function move(direction: Direction) {
  switch (direction) {
    case "up":
      console.log("Moving up");
      break;
    case "down":
      console.log("Moving down");
      break;
    case "left":
      console.log("Moving left");
      break;
    case "right":
      console.log("Moving right");
      break;
    default:
      // 一般不会进入，TypeScript 会通过穷尽检查确保覆盖所有情况
      console.error("Invalid direction");
  }
}

move("up");

// 优点：不需要额外的对象包装，书写简单。
// 缺点：仅作为类型，若希望同时具备作为值的便利（例如重复引用、迭代）则不如枚举或 const 对象灵活。
```

---

## 使用 Unique Symbol 实现名义类型枚举

在 TypeScript 中，默认采用的是基于结构的类型系统。这意味着只要对象的结构一致，类型就可以相互兼容。但有时我们希望某个类型具有名义（nominal）的特性，也就是“看名字”来判断类型而不是“看结构”。而 unique symbol 就是一种实现这一目标的工具。

```ts
// 利用 unique symbol 定义每个方向的唯一标识
const UP: unique symbol = Symbol("Up");
const DOWN: unique symbol = Symbol("Down");
const LEFT: unique symbol = Symbol("Left");
const RIGHT: unique symbol = Symbol("Right");

// 定义一个 const 对象，将每个成员映射到对应的 unique symbol
const DirectionSymbol = {
  Up: UP,
  Down: DOWN,
  Left: LEFT,
  Right: RIGHT,
} as const;

// 利用 TypeScript 的类型推导，构造一个联合类型，代表所有的枚举值
type Direction = (typeof DirectionSymbol)[keyof typeof DirectionSymbol];

/**
 * 根据传入的方向（symbol 类型）进行处理
 * @param direction - 只能是 DirectionSymbol 中的某个唯一 symbol
 */
function move(direction: Direction) {
  if (direction === DirectionSymbol.Up) {
    console.log("Moving up");
  } else if (direction === DirectionSymbol.Down) {
    console.log("Moving down");
  } else if (direction === DirectionSymbol.Left) {
    console.log("Moving left");
  } else if (direction === DirectionSymbol.Right) {
    console.log("Moving right");
  }
}

move(DirectionSymbol.Down);
```

1. **Unique Symbol 的含义**   当你声明一个变量并标记为 `unique symbol`​，TypeScript 会将该变量的类型视为一个独一无二的字面量类型，而不仅仅是普通的 `symbol`​ 类型。这就意味着即使两个变量都是 `unique symbol`​ 类型，它们各自的类型也是彼此不兼容的，除非显式地传递了同一个变量引用。
2. **如何使用**   在代码中，我们为每个枚举成员创建了一个 `const`​ 声明，并使用 `unique symbol`​ 来标记它们：

    ```typescript
    // 为每个方向声明一个 unique symbol（每个符号都具有独特的类型）
    const UP: unique symbol = Symbol("Up");
    const DOWN: unique symbol = Symbol("Down");
    const LEFT: unique symbol = Symbol("Left");
    const RIGHT: unique symbol = Symbol("Right");
    ```

    接着，我们将这些符号放入一个对象中，并使用 `as const`​ 固定其属性，确保对象属性的类型不会丢失：

    ```typescript
    // 组合成一个不可变的对象，确保各个属性的类型都是唯一的 symbol 字面量
    const DirectionSymbol = {
      Up: UP,
      Down: DOWN,
      Left: LEFT,
      Right: RIGHT,
    } as const;
    ```

    最后，通过下面这种写法：

    ```typescript
    type Direction = (typeof DirectionSymbol)[keyof typeof DirectionSymbol];
    ```

    我们就获取了 `DirectionSymbol`​ 对象中所有属性值组成的联合类型。这意味着 `Direction`​ 类型只能为 `DirectionSymbol`​ 中某个具体的成员（即某个 unique symbol）。
3. **优缺点**

    * **优点**：

      * **严格的类型保护**。由于每个 unique symbol 的类型都是独唯一的，这种方式可以防止不同枚举之间混用，达到名义类型检查的效果。
      * **类型安全**。在开发过程中，TypeScript 能够准确识别每个符号，避免一些由于结构相似而出现的类型错误。
    * **缺点**：

      * **调试和序列化问题**。运行时产生的值为 symbol，调试时输出的值不直观，而且 symbol 不能直接 JSON 序列化。
      * **不可枚举**。如果希望在运行时代码中枚举所有成员，就需要额外的工作，因为 symbol 属性不像普通的字符串那样容易遍历。

---

## 利用品牌 (Branded) 技巧实现更严格的枚举类型

在 TypeScript 中，由于采用的是结构化类型系统，即使两个类型内部结构一致也可以互相赋值。有时候这会带来隐患，比如不同业务逻辑下虽然使用的是字符串，但你不希望“green”这个字符串既代表颜色又代表其他意思。所谓“品牌化”（Branding）技术，就是通过在类型上**额外加上一个虚拟标记**，从而使得相同的基本类型在类型系统中被视为完全不同的类型。

```ts
// 定义辅助类型 ValueOf，用来提取对象中所有属性的联合类型
type ValueOf<T> = T[keyof T];

// 定义品牌包装类型，用于在原始类型上添加一个独特的标记字段
type Brand<T, B> = T & { __brand: B }; 

// 定义一个颜色映射对象，并用 as const 固定属性类型
const ColorMapping = {
  Red: "red",
  Green: "green",
  Blue: "blue",
} as const;

// 定义一个颜色类型，利用 Brand 技巧将普通的字符串类型包装为独特的 Color 类型
type Color = Brand<ValueOf<typeof ColorMapping>, "Color">;

/**
 * 一个函数，要求传入已带品牌的 Color 类型
 * @param color - 传入的颜色必须是经过品牌包装后的类型
 */
function paint(color: Color) {
  console.log(`Painting in ${color}`);
}

/**
 * 辅助函数，将对象的键转换为带品牌的 Color 类型
 * 这样可以确保只有预定义的枚举值才能得到正确的类型
 * @param key - ColorMapping 对象的键（"Red" | "Green" | "Blue"）
 * @returns 对应带品牌的 Color 类型值
 */
const makeColor = (key: keyof typeof ColorMapping): Color => {
  // 强制断言返回值为 Color (带有品牌__brand)
  return ColorMapping[key] as Color;
};

const myColor = makeColor("Green");
paint(myColor);

// 如果直接使用普通字面量会报错，例如下面这行（取消注释会产生类型错误）：
// paint("green"); 
```

1. **品牌包装**   我们定义一个辅助类型 `Brand<T, B>`​：

    ```typescript
    type Brand<T, B> = T & { __brand: B };
    ```

    其作用是将原始类型 `T`​（例如 `string`​）与一个标记属性 `__brand`​ 结合，从而得到一种新的类型。这个 `__brand`​ 字段只在类型层面存在，用于区分两个结构上完全相同的值。
2. **如何应用**   以颜色枚举为例，我们有一个对象定义了各个颜色：

    ```typescript
    const ColorMapping = {
      Red: "red",
      Green: "green",
      Blue: "blue",
    } as const;
    ```

    接着，利用 `Brand`​ 辅助类型和上面提到的提取所有属性值组成联合类型的技巧，我们定义：

    ```typescript
    // 这里 ValueOf<T> 表示取对象中所有属性值的联合类型
    type ValueOf<T> = T[keyof T];
    // 定义 Color 类型，外面加上品牌 "Color"
    type Color = Brand<ValueOf<typeof ColorMapping>, "Color">;
    ```

    这样，即使实际内容上 Color 就是字符串类型，但它携带了一个独特的品牌。  
    再使用一个辅助函数将合法的颜色转换成带品牌的类型：

    ```typescript
    const makeColor = (key: keyof typeof ColorMapping): Color => {
      return ColorMapping[key] as Color;
    };
    ```

    这样只有通过这个函数生成的值才能被视为符合 `Color`​ 类型，而直接写 `"green"`​ 则不满足品牌要求，从而避免类型混用。
3. **优缺点**

    * **优点**：

      * 利用品牌包装，能在编译阶段防止因结构一致而导致的错误赋值，增强了类型安全性。
      * 明确区分了不同业务逻辑下的相同基本类型（如字符串）的用途。
    * **缺点**：

      * 写法上比直接使用字符串或联合类型更繁琐，需要辅助函数和显式的类型断言。
      * 增加了代码的复杂度，使用者需要理解“品牌化”这一概念，可能对维护成本有所影响。

---

## 利用类型保护实现联合类型的窄化

在处理联合类型时，我们通常希望在经过某些判断后能够“窄化”（narrow）出具体的子类型，以便安全地访问对应的属性。TypeScript 内置了一些简单的窄化方式（比如 `typeof`​ 操作符或通过判别联合的 `tag`​ 字段），但有时我们需要自定义更复杂的类型保护函数。对联合类型基于标签字段（discriminated union）的检查方法。通过自定义类型保护函数，可以在运行时对值的具体形态进行判断，实现精确的类型推导。

```ts
// 定义两个接口 A 和 B，利用 type.name 字段区分
interface A {
  type: { name: "A" }; // 标签为 "A"
  a: number;
  // 此处 b 不存在（或可以声明为 undefined）
  b?: undefined;
}

interface B {
  type: { name: "B" }; // 标签为 "B"
  b: number;
  a?: undefined;
}

// 定义一个联合类型，可能是 A 或 B
const aOrB: A | B = { type: { name: "A" }, a: 42 };

/**
 * 类型保护函数，用于判断传入对象的 type.name 是否为预期的值
 * @param obj - 任意有 type 属性的对象
 * @param name - 希望匹配的名称（例如 "A" 或 "B"）
 * @returns 如果匹配则返回 true，此时 TS 可将 obj 窄化为具有特定 type 的子类型
 */
function hasTypeName<Name extends string>(
  obj: { type: { name: string } },
  name: Name
): obj is { type: { name: Name } } {
  return obj.type.name === name;
}

// 使用类型保护函数进行类型窄化
if (hasTypeName(aOrB, "A")) {
  // 在此代码块中，aOrB 已经被 TypeScript 窄化为类型 A
  console.log("It is A, property 'a' value:", aOrB.a);
} else {
  // 这里 aOrB 被窄化为类型 B
  console.log("It is B, property 'b' value:", aOrB.b);
}

// 优点：通过自定义类型保护，可以在运行时判断并窄化联合类型，方便对不同情况作出处理。
// 缺点：如果标签失效或设计不当，可能导致错误的窄化；同时在复杂场景下需要额外的辅助函数来进行判断。
```

1. **判别联合和类型保护**   当你定义多个类型且它们共享一个公共字段（例如 `type.name`​），可以利用这个字段来判断具体是哪一种情况。这就是“判别联合”（Discriminated Unions）的思想。但有时内置的判断不足以让编译器自动完成窄化，这时就需要自定义一个类型保护函数。
2. **自定义类型保护函数的写法**   自定义类型保护函数的返回值使用 `obj is SpecificType`​ 的形式，告诉编译器：如果函数返回 `true`​，那么传入的对象就可以被视为 `SpecificType`​ 类型。例如：

    ```typescript
    // 定义两个接口，通过 type.name 字段区分
    interface A {
      type: { name: "A" };
      a: number;
    }

    interface B {
      type: { name: "B" };
      b: number;
    }

    // 自定义类型保护函数：判断对象的 type.name 是否为预期值
    function hasTypeName<Name extends string>(
      obj: { type: { name: string } },
      name: Name
    ): obj is { type: { name: Name } } {
      return obj.type.name === name;
    }
    ```

    上面这个函数的返回类型 `obj is { type: { name: Name } }`​ 告诉 TypeScript，在判断返回 `true`​ 后，`obj`​ 的 `type.name`​ 一定是传入的 `name`​。这样，在 if 代码块内，就能安全地窄化出具体类型。
3. **如何使用**   假设我们有一个联合类型的变量：

    ```typescript
    const aOrB: A | B = { type: { name: "A" }, a: 42 };
    ```

    当我们使用类型保护函数进行判断时：

    ```typescript
    if (hasTypeName(aOrB, "A")) {
      // 这里 aOrB 被窄化为 A，可以安全访问属性 a
      console.log("It is A, property 'a' value:", aOrB.a);
    } else {
      // 否则，aOrB 被窄化为 B，可以安全访问属性 b
      console.log("It is B, property 'b' value:", aOrB.b);
    }
    ```

    编译器通过自定义的类型保护函数，准确推断出在不同分支中 `aOrB`​ 的具体类型，从而保证访问属性时不会出错。
4. **优缺点**

    * **优点**：

      * 自定义类型保护函数能扩展自动窄化机制，使得对联合类型的判断更灵活精确。
      * 可以在复杂条件下，通过多层判断实现精细化类型分支，提升代码的安全性。
    * **缺点**：

      * 如果设计不当（例如标签字段不唯一或不一致），可能导致类型保护失败或产生误判。
      * 需要额外定义辅助函数，增加了代码量和维护难度。

‍
