# 数字验证 Coding Guideline

## 1. 空格与缩进

- 首行2空格缩进，对齐。禁止使用tab
- 不同功能的代码间加空行
- if/while，逗号，运算符等，后面留一个空格
- 行尾不留空
- 长语句合理换行
- if/while等必须有begin/end（即使只有一行）

## 2. 注释

- 好的代码本身就是注释
- 尽量写详细，帮助自己/他人理解代码
- 建议不要加无意义的注释
- 注释掉的不再使用的代码，在环境稳定后建议删除

## 3. 变量命名

- 取有意义的变量名
- 变量名不要太长
- 使用下划线(first_second)，不要使用驼峰法(FirstSecond)
- **禁止使用global全局变量**
- 控制变量作用范围，局部变量禁止和全局变量或函数重名
- 常用前缀：局部变量m_，**句柄p_**，句柄h_，**枚举类型e_**
- **parameter和常量名大写**，变量名小写

## 4. 运算

- 使用括号控制运算优先级（==, <<, &&尤其可能混淆）
- **禁止有符号数和无符号数混合运算**

## 5. 宏

- 名称大写
- 如果定义的是运算表达式，一定要加括号
- 多行的宏首尾要加begin/end
- 控制编译范围的宏应避免在文件中define，推荐使用vcs command line:+define+
- 当作**常量使用的宏应该在统一的文件中定义**，不要定义在零散的文件中
- 非必要不使用`ifdef`，尽量使用`$testplusargs`/`$valueplusargs`等

## 6. 文件

- 直接加入filelist编译的文件后缀名为sv
- 通过include加入编译的文件后缀名为svh
- 文件名和文件中定义的class/module/interface/package同名
- 一个文件中一般只定义一种class/module/interface/package
- 文件首尾使用`ifndef保护
- 只允许在module/package内部import package
- test和sequence要取有意义的名字（避免case0/case1这种），但也不要太长
- tests.ini中每个test都要加description和owner

## 7. 打印

- 使用uvm_info打印，避免display
- 合理控制打印等级：
    - UVM_LOW (UVM_NONE): 仿真进程的关键信息等，出现次数较少
    - UVM_MEDIUM: 数据流相关信息等
    - UVM_HIGH(UVM_DEBUG): 用于debug的信息等
- 仿真verbosity: 单次仿真默认UVM_MEDIUM，回归默认UVM_LOW
- **禁止使用obj自带的print()**，使用uvm_info+sprint()

## 8. 随机化

- 所有的sequence/transaction/config，创建后都要调用randomize()，并使用if语句判断返回结果（不要用assert）
- 其他随机可以使用$urandom, $urandom_range, std::randomize()，禁止使用$random

## 9. UVM相关

- 只在test中使用configuration_phase/reset_phase/main_phase，在**其他uvm_component只使用run_phase**
- 只在test或**virtual_sequence**中raise/drop objection
- config_db::set第二个参数尽量限定路径，避免使用单一通配符*，config_db::get需要判断返回结果
- **避免绝对延时，尽量使用clock cycle，异步接口除外**
- test和virtual_sequence分别写在test_list和sequence_list里，并include在tb_top中
- **tb顶层统一命名为tb_top**，**DUT统一命名为DUT**
- **显式调用seq.start()**，避免使用default_sequence（vip除外）
- 尽量避免重复代码，重视可复用性和可移植性，多使用宏/参数/函数等

## 10. 上传相关

- 不要上传无关文件（波形，log，仿真时自动生成的文件等）
- 上传时必须要附加有意义的message
- 上传文件前必须至少跑过sanity case，同时使用cvs diff/svn diff review相关改动，防止漏传文件
