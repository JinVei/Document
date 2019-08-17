# **Lua VM** 阅读总结
##  **Chunk**
  chunk 文件是用于被 lua 虚拟机装载执行的文件，如果把lua虚拟机比成linux或者windows系统，那么chunk文件就相当于类似于 linux 下的 elf 或者 windows 下 exe 文件。chunk文件里保存了已经编译成字节码的 lua 脚本代码以及相关信息。chunk 文件结构主要由 header 字段以及 prototype 字段组成。

  **Chunk 的定义代码：**

    type binaryChunk struct {
        header       *Header
        sizeUpvalues byte
        mainFunc     *Prototype
    }
---

## **Chunck 文件里主要数据类型的结构**
  chunk文件里使用的数据类型大致可分为**数字**、**字符串**、**列表**三种。

### **1. 数字类型**
| 数据类型   | 占用字节数 |
|-|-|
| cint       | 4 |
| size_t    | 8 |
| Lua 整型   | 8 |
| Lua 浮点型 | 8 |

### **2. 字符串类型**
  字符串类型可分短字符串和长字符串两种。具体有三种情况：
  
  * 用 0x00 表示 NULL 字符串
  * 对于长度小于等于 253（0xFD）的字符串，用第一个字节记录字符串的整体长度（n+1，设 n 为字符串的长度），而之后的 n 字节用于表示字符串。
  * 对于长度大于等于254（0XFE）的字符串，第一个字节填0XFF，后接一个size_t类型的字段（8 字节）用于表示字符串的整体长度（n+1），最后用 n 个字节填上字符串数值。

### **3. 列表类型**
  列表类型有由**一个长度字段**和**n个紧凑并列的元素项**组成。长度字段的类型是cint（4字节），元素项的类型不确定，根据场景来断定。

---

## **Chunk 的 Header 字段**
  头部字段主要包含描述 chunk 一些属性信息。主要有签名、版本号、数据类型信息、效验信息等。

**Header 定义：**

    type Header struct {
        signature       [4]byte
        version         byte
        format          byte
        luacData        [6]byte
        cintSize        byte
        sizetSize       byte
        instructionSize byte
        luaIntegerSize  byte
        luaNumberSize   byte
        luacInt         int64
        luacNum         float64
    }
### **1. SIGNATURE 字段**
  签名，即所谓的的魔数，用于标识文件类型。二进制格式文件通常用头四个字节存放魔数。chunk 文件的魔数是 0X1B4C7561。

### **2. VERSION 字段**
  版本号用一个字节描述，高四位为大版本号，第四位为小版本号。如在lua5.3版本中，该版本号的值为0X53。
### **3. FORMAT 字段**
  该字段用于效验，用一个字节描述，Lua官方规定该值为0
### **4. LUAC_DATA 字段**
  该字段用于效验，用6字节描述，该字段的值为：0X19930D0A1A0A

### **5. CINT_SIZE、SIZET_SIZE、INSTRUCTION_SIZE、LUAINTEGER_SIZE、LUANUMBER_SIZE 字段**
  该字段有5个字节，分别用于描述cint类型、size_t类型、lua虚拟机指令、lua整数类型和lua浮点数类型的长度。
### **6. LUAC_INT 字段**
  该字段放一个lua整数类型的数值0X5678，用于效验大小端序。
### **7. LUAC_NUM字段**
  该字段存放一个lua浮点数类型的数值370.5，用于效验浮点数格式。lua浮点数类型格式采用 IEEE 754 浮点数格式。
  
---

## **Chunk 的 Prototype 字段**
  prototype 是用于描述函数的结构，主要有以下信息：

  * 函数的基本信息（参数数量、局部变量数量）
  * 字节码
  * 常量表
  * Upvalue表
  * 调试信息
  * 子函数原型列表

### **Prototype 定义：**
    type Prototype struct {
        source          string
        lineDefined     uint32
        lastLineDefined uint32
        paramNum        byte
        isVarargs       byte
        registerNum     byte
        instructions    []uint32
        constants       []interface{}
        upvalues        []Upvalue
        subPrototypes   []*Pototype
        lineInfo        []uint32
        localVars       []LocalVar
        upvalueNames    []string
    }
### **1. SOURCE 字段**
  该字段的类型为字符串类型，声明该函数的源文件名。
### **2. DEFINED_LINE 和 END_DEFINED_LINE 字段**
  该字段类型为cint，用于描述函数在源文件中的所在行数。
### **3. PARAMETER_NUMBER 字段**
  该字段占一字节，用于描述函数固定参数的个数。
### **4. iSVARARGS 字段**
  该字段用于描述函数是否有变长参数。
### **5. REGISTER_NUMBER 字段**
  该字段用于描述该函数用到多少寄存器。
### **6. INSTRUCTION_TABLE 字段**
  该字段用于描述指令表，指令表中每条指令占4字节。该字段的类型为列表，列表元素项的类型uint32。
### **7. CONSTANT_TABLE 字段**
  该字段类型为列表，用于描述常量表。常量表用于存放Lua代码里的字面量，包括nil、布尔值、整数、浮点数和字符串。该列表元素项以1字节tag开头，用来表示后续存储的是哪种类型的常量值。
### **8. UPVALUE_TABLE 字段**
  该字段的类型是列表，用于描述闭包中引用的变量。
### **9. SUBPROTOTYP_TABLE 字段**
  该字段的类型为列表，列表元素项的类型为Prototype，每个元素项既是一个子函数的递归定义。
### **10. LINE_INFO 字段**
  该字段用于描述行号表，行号表中记录了每条指令在源码中对应的行号。该字段类型为列表，元素项的类型cint。
### **11. LOCAL_VARIABLE_TABLE 字段**
  该字段用于描述局部变量表。该字段类型为列表，元素项的类型包含三个字段，分别是varName|string|变量名、startPC|uint32|有效区间指令索引、endPC|uint32|有效区间指令索引。
### **11. UPVALUE_NAME_TABLE 字段**
  该字段描述一个Upvalue表，该表中的元素与前面的Upvalue表中的元素一一对应，分别记录了每个upvalue在源码中的名字。

---

