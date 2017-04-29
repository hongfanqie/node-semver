semver(1) -- npm 语义化版本工具
=============================

## 安装

```bash
npm install --save semver
````

## 用法

用作模块：

```js
const semver = require('semver')

semver.valid('1.2.3') // '1.2.3'
semver.valid('a.b.c') // null
semver.clean('  =v1.2.3   ') // '1.2.3'
semver.satisfies('1.2.3', '1.x || >=2.5.0 || 5.0.0 - 7.2.3') // true
semver.gt('1.2.3', '9.8.7') // false
semver.lt('1.2.3', '9.8.7') // true
```

用作命令行工具：

```
$ semver -h

SemVer 5.3.0

A JavaScript implementation of the http://semver.org/ specification
Copyright Isaac Z. Schlueter

Usage: semver [options] <version> [<version> [...]]
Prints valid versions sorted by SemVer precedence

Options:
-r --range <range>
        Print versions that match the specified range.

-i --increment [<level>]
        Increment a version by the specified level.  Level can
        be one of: major, minor, patch, premajor, preminor,
        prepatch, or prerelease.  Default level is 'patch'.
        Only one version may be specified.

--preid <identifier>
        Identifier to be used to prefix premajor, preminor,
        prepatch or prerelease version increments.

-l --loose
        Interpret versions and ranges loosely

Program exits successfully if any valid version satisfies
all supplied ranges, and prints all satisfying versions.

If no satisfying versions are found, then exits failure.

Versions are printed in ascending order, so supplying
multiple versions to the utility will just sort them.
```

## 版本

“版本”（version）的说明见 SemVer 规范 `v2.0.0` <http://semver.org/>

版本开头的 `"="` 或 `"v"` 字符忽略并删除。

## 范围

“版本范围”（version range）是一个 “比较式” 集合，指定什么版本满足这个范围。

“比较式”（comparator）由一个操作符与一个版本组成。基本的比较式：

* `<` 小于
* `<=` 小于或等于
* `>` 大于
* `>=` 大于或等于
* `=` 等于。如果没有操作符，视为等于。因此可以省略这个操作符。

例如，比较式 `>=1.2.7` 匹配 `1.2.7`, `1.2.8`, `2.5.3`, `1.3.9`，但是不匹配
`1.2.6`, `1.1.0`。

多个比较式可以用空格连成一个比较式集合（comparator set），
在这些比较式的**交集**内的版本满足这个集合。

范围是一个比较式集合或多个用 `||` 连接起来的比较式集合。一个版本满足范围，
是当且仅当它满足了其中一个比较式集合。

例如，范围 `>=1.2.7 <1.3.0` 匹配 `1.2.7`, `1.2.8`, `1.2.99`，但是不匹配
`1.2.6`, `1.3.0`, `1.1.0`。

范围 `1.2.7 || >=1.2.9 <2.0.0` 匹配 `1.2.7`, `1.2.9`, `1.4.6`，但是不匹配
`1.2.8`,  `2.0.0`。

### 预发布标签

如果版本有预发布标签（prerelease tag，例如 `1.2.3-alpha.3`），
它只能满足这样的比较式集合：
至少其中一个比较式为相同的元组（tuple）`[major, minor, patch]` 并且包含预发布标签。

例如，对于范围 `>1.2.3-alpha.3`， `1.2.3-alpha.7` 满足，但是
`3.4.5-alpha.9` 不满足，尽管从技术上按照 SemVer 的比较规则它比
`1.2.3-alpha.3` 大，但是这个范围只接受在 `1.2.3` 上添加预发布标签。
`3.4.5` 满足，因为它没有预发布标签，并且 `3.4.5` 比 `1.2.3-alpha.7` 大。
（译注：这里添加下面表达式来帮助理解）

```javascript
semver.satisfies('1.2.3-alpha.7', '>1.2.3-alpha.3') // true
semver.satisfies('3.4.5-alpha.9', '>1.2.3-alpha.3') // false
semver.gt('3.4.5-alpha.9', '1.2.3-alpha.3')         // true
semver.satisfies('3.4.5', '>1.2.3-alpha.3')         // true
```

这样做有两个理由。一，预发布版本常常更新得比较快，并且包含许多非兼容的修改，
不适合开放使用（作者的本意如此）。因此，默认将它们排除在范围匹配之外。

二，选择使用预发布版本的用户，使用这种特殊的 alpha/beta/rc
版本清楚的表明了他们的意图。通过在范围内包含预发布标签，用户明示他们知道风险。
但是，假定他们选择继续冒险使用下一个预发布版本不合适。

#### 预发布标识符

方法 `.inc` 接受一个额外的字符串参数 `identifier`，作为预发布标识符：

```javascript
semver.inc('1.2.3', 'prerelease', 'beta')
// '1.2.4-beta.0'
```

命令行:

```bash
$ semver 1.2.3 -i prerelease --preid beta
1.2.4-beta.0
```

继续增大：

```bash
$ semver 1.2.4-beta.0 -i prerelease
1.2.4-beta.1
```

### 高级范围语法

高级范围由基本比较式以特定方式组成。

高级范围可以像基本比较式一样用空格或 `||` 组合。

#### 连字符范围（Hyphen Ranges） `X.Y.Z - A.B.C`

指定一个包含集合。

* `1.2.3 - 2.3.4` := `>=1.2.3 <=2.3.4`

如果第一个版本格式不完整，则它缺失的部分补为 0。

* `1.2 - 2.3.4` := `>=1.2.0 <=2.3.4`

如果第二个版本格式不完整，则以它开头的版本都接受。

* `1.2.3 - 2.3` := `>=1.2.3 <2.4.0`
* `1.2.3 - 2` := `>=1.2.3 <3.0.0`

#### X 范围（X-Ranges） `1.2.x` `1.X` `1.2.*` `*`

用 `X`, `x` 或 `*` 代表元组 `[major, minor, patch]` 的某部分。

* `*` := `>=0.0.0` （任意版本均满足）
* `1.x` := `>=1.0.0 <2.0.0` （匹配主版本）
* `1.2.x` := `>=1.2.0 <1.3.0` （匹配主版本与副版本）

格式不完整的版本按 X 范围处理，所以实际上可以省略字符 X。

* `""` （空字符串）:= `*` := `>=0.0.0`
* `1` := `1.x.x` := `>=1.0.0 <2.0.0`
* `1.2` := `1.2.x` := `>=1.2.0 <1.3.0`

#### 波浪号范围（Tilde Ranges） `~1.2.3` `~1.2` `~1`

比较式如果有副版本，允许改变修订版本；如果没有，允许改变副版本。

* `~1.2.3` := `>=1.2.3 <1.(2+1).0` := `>=1.2.3 <1.3.0`
* `~1.2` := `>=1.2.0 <1.(2+1).0` := `>=1.2.0 <1.3.0` （同 `1.2.x`）
* `~1` := `>=1.0.0 <(1+1).0.0` := `>=1.0.0 <2.0.0` （同 `1.x`）
* `~0.2.3` := `>=0.2.3 <0.(2+1).0` := `>=0.2.3 <0.3.0`
* `~0.2` := `>=0.2.0 <0.(2+1).0` := `>=0.2.0 <0.3.0` （同 `0.2.x`）
* `~0` := `>=0.0.0 <(0+1).0.0` := `>=0.0.0 <1.0.0` （同 `0.x`）
* `~1.2.3-beta.2` := `>=1.2.3-beta.2 <1.3.0` 注意 `1.2.3` 可以有预发布标签，
  只要标签大于或等于 `beta.2`。因此 `1.2.3-beta.4` 满足，但是
  `1.2.4-beta.2` 不满足，因为它是不同的元组 `[major, minor, patch]` 的
  预发布标签。

#### 脱字符范围（Caret Ranges） `^1.2.3` `^0.2.5` `^0.0.4`

不能改变元组 `[major, minor, patch]` 最左边的非零数字。例如，`1.0.0`
可以更新修订版本与副版本，`0.X >=0.1.0` 可以更新修订版本，`0.0.X` 不能更新。

许多作者将 `0.x` 中的 `x` 当作非兼容更新的标志。

当作者在 `0.2.4` 与 `0.3.0` 之间作了非兼容更新，这时使用脱字符范围比较理想，
这也是常见做法。不过这假定了从 `0.2.4` 到 `0.2.5` **不是**非兼容更新，
以用于兼容性更新。

* `^1.2.3` := `>=1.2.3 <2.0.0`
* `^0.2.3` := `>=0.2.3 <0.3.0`
* `^0.0.3` := `>=0.0.3 <0.0.4`
* `^1.2.3-beta.2` := `>=1.2.3-beta.2 <2.0.0` 注意 `1.2.3` 可以有预发布标签，
  只要标签大于或等于 `beta.2`。因此，`1.2.3-beta.4` 满足，但是
  `1.2.4-beta.2` 不满足，因为它是不同的元组 `[major, minor, patch]` 的
  预发布标签。
* `^0.0.3-beta` := `>=0.0.3-beta <0.0.4` 注意只能用 `0.0.3` 预发布版本，只要
  标签大于或等于 `beta`。因此，`0.0.3-pr.2` 不满足。

在解析脱字符范围时，缺失的 `patch` 补为 0，不过它可以变，
即使主版本与副版本都为 `0`。

* `^1.2.x` := `>=1.2.0 <2.0.0`
* `^0.0.x` := `>=0.0.0 <0.1.0`
* `^0.0` := `>=0.0.0 <0.1.0`

`minor` 和 `patch` 都缺失时都补为 0，同样的两者都可以变，即使主版本为 `0`。

* `^1.x` := `>=1.0.0 <2.0.0`
* `^0.x` := `>=0.0.0 <1.0.0`

### 范围语法

为方便解析器作者，在下面列出范围的 Backus-Naur 语法：

```bnf
range-set  ::= range ( logical-or range ) *
logical-or ::= ( ' ' ) * '||' ( ' ' ) *
range      ::= hyphen | simple ( ' ' simple ) * | ''
hyphen     ::= partial ' - ' partial
simple     ::= primitive | partial | tilde | caret
primitive  ::= ( '<' | '>' | '>=' | '<=' | '=' | ) partial
partial    ::= xr ( '.' xr ( '.' xr qualifier ? )? )?
xr         ::= 'x' | 'X' | '*' | nr
nr         ::= '0' | ['1'-'9'] ( ['0'-'9'] ) *
tilde      ::= '~' partial
caret      ::= '^' partial
qualifier  ::= ( '-' pre )? ( '+' build )?
pre        ::= parts
build      ::= parts
parts      ::= part ( '.' part ) *
part       ::= nr | [-0-9A-Za-z]+
```

## 函数

所有的方法与类都可以接受最后一个参数 `loose`，如果它为 `true`,
可以解析不怎么严格的版本。当然处理的结果始终是 100% 严格的语义化版本。

严格模式的比较式与范围严格地解析版本。

* `valid(v)`: 返回解析后的版本，无效的返回 `null`。
* `inc(v, release)`: 根据 release 类型（`major`, `premajor`,
  `minor`, `preminor`, `patch`, `prepatch`, 或 `prerelease`）返回增大的版本，
  无效的返回 `null`。
  * `premajor` 增大到下一个主版本，然后生成一个预发布版本。
    `preminor` 与 `prepatch` 同理。
  * 如果不是预发布版本，`prerelease` 的效果同 `prepatch`，
    增大修订版本，然后生成一个预发布版本。如果已是预发布版本，只增大修订版本。
* `prerelease(v)`: 返回一个数组，包含预发布版本组成部分。如果预发布版本则返回 null。
  例如： `prerelease('1.2.3-alpha.1') -> ['alpha', 1]`
* `major(v)`: 返回主版本。
* `minor(v)`: 返回副版本。
* `patch(v)`: 返回修订版本。

### 比较

* `gt(v1, v2)`: `v1 > v2`
* `gte(v1, v2)`: `v1 >= v2`
* `lt(v1, v2)`: `v1 < v2`
* `lte(v1, v2)`: `v1 <= v2`
* `eq(v1, v2)`: `v1 == v2` 如果它们逻辑上相等则结果为 `true`，即使它们不是
  相同的字符串。你知道怎么比较字符串。
* `neq(v1, v2)`: `v1 != v2` 是 `eq` 的否定。
* `cmp(v1, comparator, v2)`: 传入一个比较符，调用上述相应的函数。
  `"==="` 和 `"!=="` 只是简单的对比字符串，包含它们只是为了完整性。
  如果传入无效的比较符则抛出异常。
* `compare(v1, v2)`: 如果 `v1 == v2` 返回 `0`, 如果 `v1` 更大返回 `1`,
  如果 `v2` 更大返回 `-1`。如果传入 `Array.sort()` 则数组升序排列。
* `rcompare(v1, v2)`: 颠倒 `compare` 的结果。如果传入 `Array.sort()` 则数组降序排列。
* `diff(v1, v2)`: 根据 release 的类型（`major`, `premajor`, `minor`, `preminor`,
  `patch`, `prepatch` 或 `prerelease`）返回两个版本的不同部分。
  如果两者相同返回 null。

### 范围

* `validRange(range)`: 返回有效的范围，无效的返回 `null`。
* `satisfies(version, range)`: 返回 `true` 若版本满足范围。
* `maxSatisfying(versions, range)`: 返回所有满足范围的版本当中最大的一个，
  如果没有则返回 `null`。
* `minSatisfying(versions, range)`: 返回所有满足范围的版本当中最小的一个，
  如果没有则返回 `null`。
* `gtr(version, range)`: 返回 `true` 若版本大于所有满足范围的版本。
* `ltr(version, range)`: 返回 `true` 若版本小于所有满足范围的版本。
* `outside(version, range, hilo)`: 返回 `true` 若版本在范围的边界外。
  参数 `hilo` 必须是 `'>'` 或 `'<'`，它会调用相应的函数 `gtr` 或 `ltr`。

注意，范围可能是不连续的，因而版本可能既不大于范围，也不小于范围，也不满足范围！
例如，范围 `1.2 <1.2.9 || >2.0.0` 缺漏 `1.2.9` 到 `2.0.0`，`1.2.10` 不大于
范围（更高的 `2.0.1` 满足），不小于范围（更低的 `1.2.8` 满足），也不满足范围。

如果你想知道版本是否满足范围，使用 `satisfies(version, range)` 测试。

## 翻译

[本文](https://github.com/npm/node-semver#readme) 由 [Ivan Yan](http://yanxyz.net/) 翻译，
译文采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>。

译文[更新](update.md)，意见[反馈](https://github.com/hongfanqie/node-semver/issues)。
