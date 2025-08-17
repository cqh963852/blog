# 关于 Lisp 方言的想法

在过去，现在，我都很喜欢 Lisp。虽然他的括号看起来那么烦。

最近又想起了 DaDa-Lang，虽然这个语言并未实现，但是有很多新奇的概念。

我也想写一个这样的概念型语言，只是为了好玩

## 这个方言要有“所有权”

所有权这个概念中，我了解的最多的是 Rust。写过一些单线程下的代码，整体概念并不难，但是我觉得还是略微繁琐，所以我还是想改动一下。

在这个方言中，我要假设，所有的内存都是“租”来的。编译器才是申请内存，释放内存的“人”。

```lisp
;; 租到 i32 大小的内存
(let a (i32))
```

所以要引入租约的概念，声明一个变量，就是声明了一个租约。这就表示，这块内存归“我”了。

那么是否可以“合租”呢？我觉得是可以的。但是这个租约不太一样，有多少人一起租得写清楚。

```lisp
;; 两位合租一个 i32 大小内存
(lets 2 a (i32))
(lets 2 b a)
```

不过也有可能不知道要几位合租，这里也可以用任意字符串来辅助

```lisp
;; 合租一个 i32 大小内存。鼠标悬浮在 n 上，会告诉你当前是几位合租。
(lets 'n' a (i32))
(lets 'n' b a)
```

## 我有一个“大宝贝”

我租来了一个宝贝，有人想看看，有人想...

我要给他看吗？他只是看看，还是要动我的宝贝？他会还给我吗？

那既然这样，那得写清楚，是分享了这个租赁，还是转移了这个租赁.

- 如果期望使用这块内存，再也不归还，那么是转移租赁。

  ```lisp
  (let a (i32))
  (relet b a)
  ;; error ，a 已经没有租约了，不能够访问内存
  (print a)
  ```

- 如果期望使用完，会归还，要使用分享租赁

  ```lisp
  (let a (i32))
  (sublet b a)
  ;; 可以输出 a
  (print a)
  ```

## 命运多舛小汽车

车行拥有租赁权，不直接向编译器获取租约

- 我想要租一辆车。

  ```ts
  (defun create_car () (relet Car)
    (let car (Car))
    (return car)
  )

  (relet a (Car) (create_car))
  ```

- 我想要和朋友们合租一辆车

  ```ts
  (defun create_car () (relets 'n' Car)
    (let car (Car))
    (return car)
  )

  (relets 2 a (Car) (create_car))
  (relets 1 b a)
  ```

- 我租了一辆车，想要送去保养，所以我要将车临时交给别人一段时间。

  ```ts
  (defun maintenance (sublet car (Car))
    (car.clean)
    (car.refuel)
    (car.park)
    (return car)
  )

  (let a (Car))
  (maintenance a)
  (a.drive)
  ```

- 我租了一辆车，但是我不想要了，我转移了租赁合同给另外一个人，并且我重新租了车

  ```ts
  (defun sell (reblet car (Car)))

  (let a (Car))
  (sell a)

  (let a (Car) (create_car))
  (a.drive)
  ```

- 有一段时间我比较忙，车子的轮胎坏了，需要开着车同时，维修轮胎。

  ```ts
  (defun fix_at_runtime (car (sublets 1 Car))
    (spawn (lambda ()
      (loop
        (await (car.fix))
      )
    ))
  )
  (let a (Car))
  (fix_at_runtime a)
  (loop (a.run))
  ```

## 单租、合租、转移租赁、分享租赁的进一步说明

### 单租

`let`

单租：内存只属于一个变量。

```ts
(let a (i32))
```

其他人无法直接访问。

### 合租

`(lets string var type)`
Lisp 风格：字符串标识合租人数，例如 `(lets "n" a (i32))`

`(lets number var type)`
Lisp 风格：数字标识合租人数，例如 `(lets 2 a (i32))`

合租允许多个变量共同拥有一块内存，可以同时读写。

例如：两个变量共有 i32

```ts
(lets 2 a (i32))
(lets 2 b a)
```

例如：编译器自动计算`‘n’`个变量共有 i32

```ts
(lets 'n' a (i32))
(lets 'n' b a)
```

### 转移租赁

`(relet var src)`
Lisp 风格：转移租赁，例如 `(relet b a)`

`(relets string var type)`
Lisp 风格：字符串标识转移合租，例如 `(relets "n" a (i32))`

`(relets number var type)`
Lisp 风格：数字标识转移合租，例如 `(relets 2 a (i32))`

转移租赁后与原变量无关

### 分享租赁

`(sublet var src)`
Lisp 风格：分享租赁，例如 `(sublet b a)`

`(sublets string var type)`
Lisp 风格：字符串标识分享合租，例如 `(sublets "n" a (i32))`

`(sublets number var type)`
Lisp 风格：数字标识分享合租，例如 `(sublets 2 a (i32))`

只有一个租户能带走猫，避免猫被一群人抢着养，编译器会帮你看门！

1. let
2. relet
3. 只有一个租户的 lets
4. 只有一个租户的 relets
