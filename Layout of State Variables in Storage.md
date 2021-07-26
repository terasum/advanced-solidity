# Storage 中的状态变量布局



契约的状态变量以紧凑的方式存储在存储器中，使得多个值有时使用同一存储槽。除了动态大小的数组和映射（见下文）之外，数据是以第一个状态变量开始的一项接一项地连续存储的，第一个状态变量存储在槽“0”中。对于每个变量，以字节为单位的大小是根据其类型确定的。如果可能，根据以下规则，需要少于32字节的多个连续项将打包到单个存储插槽中：

State variables of contracts are stored in storage in a compact way such that multiple values sometimes use the same storage slot. Except for dynamically-sized arrays and mappings (see below), data is stored contiguously item after item starting with the first state variable, which is stored in slot `0`. For each variable, a size in bytes is determined according to its type. Multiple, contiguous items that need less than 32 bytes are packed into a single storage slot if possible, according to the following rules:

- The first item in a storage slot is stored lower-order aligned.
- -存储槽中的第一个项目以较低的顺序存储。
- Value types use only as many bytes as are necessary to store them.
- 值类型只使用存储它们所需的字节数。
- If a value type does not fit the remaining part of a storage slot, it is stored in the next storage slot.
- 如果值类型不适合存储插槽的剩余部分，则它将存储在下一个存储插槽中。
- Structs and array data always start a new slot and their items are packed tightly according to these rules.
- -结构和数组数据总是开始一个新的槽，并且它们的项是根据这些规则紧密打包的。
- Items following struct or array data always start a new storage slot.
- 结构或数组数据后面的项总是启动一个新的存储插槽。

For contracts that use inheritance, the ordering of state variables is determined by the C3-linearized order of contracts starting with the most base-ward contract. If allowed by the above rules, state variables from different contracts do share the same storage slot.

对于使用继承的契约，状态变量的顺序由从最基本的ward契约开始的C3线性化契约顺序决定。如果上述规则允许，来自不同契约的状态变量确实共享同一个存储槽。

The elements of structs and arrays are stored after each other, just as if they were given as individual values.

结构和数组的元素是一个接一个地存储的，就像它们作为单独的值被给定一样。

Warning

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

当使用小于32字节的元素时，合同的气体使用量可能会更高。这是因为EVM一次操作32个字节。因此，如果元素小于该值，EVM必须使用更多的操作，以便将元素的大小从32字节减少到所需的大小。

It might be beneficial to use reduced-size types if you are dealing with storage values because the compiler will pack multiple elements into one storage slot, and thus, combine multiple reads or writes into a single operation. If you are not reading or writing all the values in a slot at the same time, this can have the opposite effect, though: When one value is written to a multi-value storage slot, the storage slot has to be read first and then combined with the new value such that other data in the same slot is not destroyed.

如果您处理的是存储值，那么使用缩减大小的类型可能是有益的，因为编译器会将多个元素打包到一个存储槽中，从而将多个读取或写入合并到一个操作中。如果不是同时读取或写入插槽中的所有值，则会产生相反的效果：当一个值写入多值存储插槽时，必须先读取该存储插槽，然后再与新值组合，以便不会破坏同一插槽中的其他数据。

When dealing with function arguments or memory values, there is no inherent benefit because the compiler does not pack these values.

在处理函数参数或内存值时，没有固有的好处，因为编译器不打包这些值。

Finally, in order to allow the EVM to optimize for this, ensure that you try to order your storage variables and `struct` members such that they can be packed tightly. For example, declaring your storage variables in the order of `uint128, uint128, uint256` instead of `uint128, uint256, uint128`, as the former will only take up two slots of storage whereas the latter will take up three.

最后，为了允许EVM对此进行优化，请确保您尝试对存储变量和“struct”成员进行排序，以便它们可以紧密打包。例如，按“uint128，uint128，uint256”的顺序声明存储变量，而不是按“uint128，uint256，uint128”的顺序声明，因为前者只占用两个存储槽，而后者占用三个存储槽。

Note

The layout of state variables in storage is considered to be part of the external interface of Solidity due to the fact that storage pointers can be passed to libraries. This means that any change to the rules outlined in this section is considered a breaking change of the language and due to its critical nature should be considered very carefully before being executed.

由于存储指针可以传递给库，因此存储中状态变量的布局被认为是稳定的外部接口的一部分。这意味着，对本节所述规则的任何更改都被视为对语言的破坏性更改，由于其批评性质，在执行之前应非常仔细地考虑。

## Mappings and Dynamic Arrays

Due to their unpredictable size, mappings and dynamically-sized array types cannot be stored “in between” the state variables preceding and following them. Instead, they are considered to occupy only 32 bytes with regards to the [rules above](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-inplace-encoding) and the elements they contain are stored starting at a different storage slot that is computed using a Keccak-256 hash.



由于其不可预测的大小，映射和动态大小的数组类型不能存储在它们前面和后面的状态变量之间。相反，根据[上述规则]它们被认为只占用32字节(https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-它们所包含的元素从使用Keccak-256散列计算的不同存储槽开始存储。





Assume the storage location of the mapping or array ends up being a slot `p` after applying [the storage layout rules](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-inplace-encoding). For dynamic arrays, this slot stores the number of elements in the array (byte arrays and strings are an exception, see [below](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#bytes-and-string)). For mappings, the slot stays empty, but it is still needed to ensure that even if there are two mappings next to each other, their content ends up at different storage locations.

假设在应用[存储布局规则]之后，映射或数组的存储位置最终是一个slot`p`(https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-就地编码）。对于动态数组，此插槽存储数组中的元素数（字节数组和字符串是一个例外，请参见[下文](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#bytes-和字符串）。对于映射，插槽保持为空，但仍然需要确保即使有两个映射相邻，它们的内容也会在不同的存储位置结束。





Array data is located starting at `keccak256(p)` and it is laid out in the same way as statically-sized array data would: One element after the other, potentially sharing storage slots if the elements are not longer than 16 bytes. Dynamic arrays of dynamic arrays apply this rule recursively. The location of element `x[i][j]`, where the type of `x` is `uint24[][]`, is computed as follows (again, assuming `x` itself is stored at slot `p`): The slot is `keccak256(keccak256(p) + i) + floor(j / floor(256 / 24))` and the element can be obtained from the slot data `v` using `(v >> ((j % floor(256 / 24)) * 24)) & type(uint24).max`.

数组数据的位置从“keccak256（p）”开始，其布局方式与静态大小的数组数据相同：一个元素接一个，如果元素长度不超过16字节，则可能共享存储槽。动态数组的动态数组递归地应用这个规则。元素“x[i][j]`”的位置，其中“x”的类型是“uint24[][]”，计算如下（同样，假设“x”本身存储在插槽“p`”）：插槽是“keccak256（keccak256（p）+i）+floor（j/floor（256/24））`并且可以使用`（v>>（（j%floor（256/24））*24）和type（uint24）.max`，从插槽数据“v”中获取元素。





The value corresponding to a mapping key `k` is located at `keccak256(h(k) . p)` where `.` is concatenation and `h` is a function that is applied to the key depending on its type:

对应于映射键“k”的值位于“keccak256（h（k））。p） 其中，`.`是串联，`h`是根据键的类型应用于键的函数：

- for value types, `h` pads the value to 32 bytes in the same way as when storing the value in memory.
- -对于值类型，`h`将值填充到32字节，方式与在内存中存储值时相同。
- for strings and byte arrays, `h` computes the `keccak256` hash of the unpadded data.
- 对于字符串和字节数组，“h”计算未添加数据的“keccak256”哈希。

If the mapping value is a non-value type, the computed slot marks the start of the data. If the value is of struct type, for example, you have to add an offset corresponding to the struct member to reach the member.

如果映射值是非值类型，则计算的槽将标记数据的开始。例如，如果值是struct类型，则必须添加与struct成员对应的偏移量才能到达该成员。

As an example, consider the following contract:

作为一个例子，考虑以下合同：

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.0 <0.9.0;


contract C {
    struct S { uint16 a; uint16 b; uint256 c; }
    uint x;
    mapping(uint => mapping(uint => S)) data;
}
```

Let us compute the storage location of `data[4][9].c`. The position of the mapping itself is `1` (the variable `x` with 32 bytes precedes it). This means `data[4]` is stored at `keccak256(uint256(4) . uint256(1))`. The type of `data[4]` is again a mapping and the data for `data[4][9]` starts at slot `keccak256(uint256(9) . keccak256(uint256(4) . uint256(1)))`. The slot offset of the member `c` inside the struct `S` is `1` because `a` and `b` are packed in a single slot. This means the slot for `data[4][9].c` is `keccak256(uint256(9) . keccak256(uint256(4) . uint256(1))) + 1`. The type of the value is `uint256`, so it uses a single slot.

让我们计算'data[4][9].c'的存储位置。映射本身的位置是“1”（前面有32个字节的变量“x”）。这意味着“数据[4]”存储在“keccak256（uint256（4）”中。uint256（1））`。`data[4]`的类型也是一种映射，`data[4][9]`的数据从插槽`keccak256（uint256（9））开始。keccak256（uint256（4）。uint256（1））`。结构“S”中成员“c”的槽偏移量为“1”，因为“a”和“b”打包在单个槽中。这意味着'data[4][9].c'的槽是'keccak256（uint256（9））。keccak256（uint256（4）。uint256（1）））+1`。值的类型为“uint256”，因此它使用单个插槽。





### `bytes` and `string`

`bytes` and `string` are encoded identically. In general, the encoding is similar to `bytes1[]`, in the sense that there is a slot for the array itself and a data area that is computed using a `keccak256` hash of that slot’s position. However, for short values (shorter than 32 bytes) the array elements are stored together with the length in the same slot.

`字节和字符串的编码相同。一般来说，这种编码与“bytes1[]”类似，因为数组本身有一个槽，数据区是使用该槽位置的“keccak256”散列计算的。但是，对于短值（小于32字节），数组元素与长度一起存储在同一插槽中。

In particular: if the data is at most `31` bytes long, the elements are stored in the higher-order bytes (left aligned) and the lowest-order byte stores the value `length * 2`. For byte arrays that store data which is `32` or more bytes long, the main slot `p` stores `length * 2 + 1` and the data is stored as usual in `keccak256(p)`. This means that you can distinguish a short array from a long array by checking if the lowest bit is set: short (not set) and long (set).

特别是：如果数据长度最多为'31'字节，则元素以高阶字节（左对齐）存储，低阶字节存储值'length*2'。对于存储长度为'32'或以上字节的数据的字节数组，主槽'p'存储长度为'length*2+1'，数据通常存储在'keccak256（p）`中。这意味着您可以通过检查是否设置了最低位来区分短数组和长数组：short（未设置）和long（设置）。

Note

Handling invalidly encoded slots is currently not supported but may be added in the future. If you are compiling via the experimental IR-based compiler pipeline, reading an invalidly encoded slot results in a `Panic(0x22)` error.

目前不支持处理无效编码的插槽，但将来可能会添加。如果您是通过实验性的基于IR的编译器管道进行编译，那么读取无效编码的插槽将导致“Panic（0x22）”错误。

## JSON Output

The storage layout of a contract can be requested via the [standard JSON interface](https://docs.soliditylang.org/en/latest/using-the-compiler.html#compiler-api). The output is a JSON object containing two keys, `storage` and `types`. The `storage` object is an array where each element has the following form:

契约的存储布局可以通过[standard JSON interface]请求(https://docs.soliditylang.org/en/latest/using-the-compiler.html#compiler-api）。输出是一个JSON对象，包含两个键“storage”和“types”。“storage”对象是一个数组，其中每个元素的形式如下：

```
{
    "astId": 2,
    "contract": "fileA:A",
    "label": "x",
    "offset": 0,
    "slot": "0",
    "type": "t_uint256"
}
```

The example above is the storage layout of `contract A { uint x; }` from source unit `fileA` and

上面的示例是“contract A{uint x；}”的存储布局从源单元“fileA”和

- `astId` is the id of the AST node of the state variable’s declaration
- `astId`是状态变量声明的AST节点的id
- `contract` is the name of the contract including its path as prefix
- `contract`是合同的名称，包括其路径作为前缀
- `label` is the name of the state variable
- `label`是状态变量的名称
- `offset` is the offset in bytes within the storage slot according to the encoding
- `offset`是存储槽中根据编码的字节偏移量
- `slot` is the storage slot where the state variable resides or starts. This number may be very large and therefore its JSON value is represented as a string.
- -`slot`是状态变量驻留或启动的存储插槽。这个数字可能非常大，因此它的JSON值被表示为一个字符串。
- `type` is an identifier used as key to the variable’s type information (described in the following)
- -“type”是用作变量类型信息键的标识符（如下所述）

The given `type`, in this case `t_uint256` represents an element in `types`, which has the form:

给定的“type”（在本例中为“t\u uint256”）表示“types”中的一个元素，其形式如下：

```
{
    "encoding": "inplace",
    "label": "uint256",
    "numberOfBytes": "32",
}
```

where

- `encoding` how the data is encoded in storage, where the possible values are:
  - `inplace`: data is laid out contiguously in storage (see [above](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-inplace-encoding)).
  - `mapping`: Keccak-256 hash-based method (see [above](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-hashed-encoding)).
  - `dynamic_array`: Keccak-256 hash-based method (see [above](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-hashed-encoding)).
  - `bytes`: single slot or Keccak-256 hash-based depending on the data size (see [above](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#bytes-and-string)).
- `label` is the canonical type name.
- `numberOfBytes` is the number of used bytes (as a decimal string). Note that if `numberOfBytes > 32` this means that more than one slot is used.

Some types have extra information besides the four above. Mappings contain its `key` and `value` types (again referencing an entry in this mapping of types), arrays have its `base` type, and structs list their `members` in the same format as the top-level `storage` (see [above](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-layout-top-level)).



哪里
-`encoding`数据在存储器中的编码方式，其中可能的值为：
-‘就地’：数据在存储器中连续排列（见[上](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-就地编码）。
-`mapping`:Keccak-256基于哈希的方法（见[上文](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-哈希编码）。
-`dynamic_array`:Keccak-256基于哈希的方法（见[上](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-哈希编码）。
-`bytes`：基于数据大小的单槽或Keccak-256散列（见上文）(https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#bytes-和字符串）。
-“label”是规范类型名。
-`numberOfBytes`是已使用的字节数（作为十进制字符串）。请注意，如果“numberOfBytes>32”，则表示使用了多个插槽。
除了上述四种类型之外，有些类型还有额外的信息。映射包含其“key”和“value”类型（再次引用此类型映射中的条目），数组具有其“base”类型，结构以与顶级“storage”相同的格式列出其“members”（参见[上文](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#storage-布局顶层）。





Note

The JSON output format of a contract’s storage layout is still considered experimental and is subject to change in non-breaking releases of Solidity.

The following example shows a contract and its storage layout, containing value and reference types, types that are encoded packed, and nested types.

合同存储布局的JSON输出格式仍然被认为是实验性的，并且在稳定的非破坏性版本中会发生变化。

下面的示例显示协定及其存储布局，其中包含值和引用类型、编码打包的类型和嵌套类型。



```
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.0 <0.9.0;
contract A {
    struct S {
        uint128 a;
        uint128 b;
        uint[2] staticArray;
        uint[] dynArray;
    }

    uint x;
    uint y;
    S s;
    address addr;
    mapping (uint => mapping (address => bool)) map;
    uint[] array;
    string s1;
    bytes b1;
}
{
  "storage": [
    {
      "astId": 15,
      "contract": "fileA:A",
      "label": "x",
      "offset": 0,
      "slot": "0",
      "type": "t_uint256"
    },
    {
      "astId": 17,
      "contract": "fileA:A",
      "label": "y",
      "offset": 0,
      "slot": "1",
      "type": "t_uint256"
    },
    {
      "astId": 20,
      "contract": "fileA:A",
      "label": "s",
      "offset": 0,
      "slot": "2",
      "type": "t_struct(S)13_storage"
    },
    {
      "astId": 22,
      "contract": "fileA:A",
      "label": "addr",
      "offset": 0,
      "slot": "6",
      "type": "t_address"
    },
    {
      "astId": 28,
      "contract": "fileA:A",
      "label": "map",
      "offset": 0,
      "slot": "7",
      "type": "t_mapping(t_uint256,t_mapping(t_address,t_bool))"
    },
    {
      "astId": 31,
      "contract": "fileA:A",
      "label": "array",
      "offset": 0,
      "slot": "8",
      "type": "t_array(t_uint256)dyn_storage"
    },
    {
      "astId": 33,
      "contract": "fileA:A",
      "label": "s1",
      "offset": 0,
      "slot": "9",
      "type": "t_string_storage"
    },
    {
      "astId": 35,
      "contract": "fileA:A",
      "label": "b1",
      "offset": 0,
      "slot": "10",
      "type": "t_bytes_storage"
    }
  ],
  "types": {
    "t_address": {
      "encoding": "inplace",
      "label": "address",
      "numberOfBytes": "20"
    },
    "t_array(t_uint256)2_storage": {
      "base": "t_uint256",
      "encoding": "inplace",
      "label": "uint256[2]",
      "numberOfBytes": "64"
    },
    "t_array(t_uint256)dyn_storage": {
      "base": "t_uint256",
      "encoding": "dynamic_array",
      "label": "uint256[]",
      "numberOfBytes": "32"
    },
    "t_bool": {
      "encoding": "inplace",
      "label": "bool",
      "numberOfBytes": "1"
    },
    "t_bytes_storage": {
      "encoding": "bytes",
      "label": "bytes",
      "numberOfBytes": "32"
    },
    "t_mapping(t_address,t_bool)": {
      "encoding": "mapping",
      "key": "t_address",
      "label": "mapping(address => bool)",
      "numberOfBytes": "32",
      "value": "t_bool"
    },
    "t_mapping(t_uint256,t_mapping(t_address,t_bool))": {
      "encoding": "mapping",
      "key": "t_uint256",
      "label": "mapping(uint256 => mapping(address => bool))",
      "numberOfBytes": "32",
      "value": "t_mapping(t_address,t_bool)"
    },
    "t_string_storage": {
      "encoding": "bytes",
      "label": "string",
      "numberOfBytes": "32"
    },
    "t_struct(S)13_storage": {
      "encoding": "inplace",
      "label": "struct A.S",
      "members": [
        {
          "astId": 3,
          "contract": "fileA:A",
          "label": "a",
          "offset": 0,
          "slot": "0",
          "type": "t_uint128"
        },
        {
          "astId": 5,
          "contract": "fileA:A",
          "label": "b",
          "offset": 16,
          "slot": "0",
          "type": "t_uint128"
        },
        {
          "astId": 9,
          "contract": "fileA:A",
          "label": "staticArray",
          "offset": 0,
          "slot": "1",
          "type": "t_array(t_uint256)2_storage"
        },
        {
          "astId": 12,
          "contract": "fileA:A",
          "label": "dynArray",
          "offset": 0,
          "slot": "3",
          "type": "t_array(t_uint256)dyn_storage"
        }
      ],
      "numberOfBytes": "128"
    },
    "t_uint128": {
      "encoding": "inplace",
      "label": "uint128",
      "numberOfBytes": "16"
    },
    "t_uint256": {
      "encoding": "inplace",
      "label": "uint256",
      "numberOfBytes": "32"
    }
  }
}
```