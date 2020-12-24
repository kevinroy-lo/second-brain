## what
转换是改变代码行为的简单操作
> Transformations are simple operations that change the behavior of code


这个理论为 [[TDD]] 提供了更加明确细节的操作说明

其作用是：将代码的行为从特定的行为转换为更通用的行为。

### vs重构

[[重构]]是在不改变代码行为的情况下改变代码结构的简单操作

## how
- 在通过测试时，更喜欢优先级更高的转换
- 当写测试时，选择一个可以通过更高优先级转换的测试
- 当一个实现似乎需要低优先级的转换时，回溯看看是否有一个更简单的测试可以通过

### The Transformations
转换列表，从简单到困难

箭头所指就是转换的方向所在

- ({}–>nil) no code at all->code that employs nil ({}-> nil) 
- (nil->constant) (nil-> 常数)
- (constant->constant+) a simple constant to a more complex constant
- (constant->scalar) replacing a constant with a variable or an argument
- (statement->statements) adding more unconditional statements.
- (unconditional->if) splitting the execution path
- (scalar->array) 
- (array->container) 
- (statement->recursion)
- (if->while)
- (expression->function) replacing an expression with a function or algorithm
- (variable->assignment) replacing the value of a variable.

## 参考资料
- [TheTransformationPriorityPremise](https://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html)