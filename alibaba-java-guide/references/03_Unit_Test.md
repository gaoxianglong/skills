# 第三章 单元测试

1.  **【强制】** 好的单元测试必须遵守 AIR 原则：
    *   A: Automatic (自动化)
    *   I: Independent (独立性)
    *   R: Repeatable (可重复)
2.  **【强制】** 单元测试应该是全自动执行的，并且非交互式的。
3.  **【强制】** 单元测试是可以重复执行的，不能受外界环境的影响。
4.  **【推荐】** 单元测试代码必须写在如下工程目录：`src/test/java`，不允许写在业务代码目录下。
5.  **【推荐】** 编写单元测试代码遵守 BCDE 原则：
    *   B: Border，边界值测试，包括循环边界、特殊取值、特殊时间点、数据顺序等。
    *   C: Correct，正确的输入，并得到预期的结果。
    *   D: Design，与设计文档相结合，来编写单元测试。
    *   E: Error，强制错误信息输入（如：非法数据、异常流程、非业务允许输入等），并得到预期的结果。
    
    ```java
    // 示例：JUnit 5 单元测试
    import org.junit.jupiter.api.Test;
    import static org.junit.jupiter.api.Assertions.assertEquals;

    public class CalculatorTest {
        
        @Test
        public void testAdd() {
            Calculator calculator = new Calculator();
            // Correct: 预期结果
            assertEquals(5, calculator.add(2, 3));
            
            // Border: 边界值
            assertEquals(Integer.MAX_VALUE, calculator.add(Integer.MAX_VALUE, 0));
        }
    }
    ```
