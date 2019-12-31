# Java8 指导

## 接口的默认方法
java8能够使用default关键词向接口添加非抽象方法实现。虚拟扩展方法
例1：

    interface Formula {
        double calculate(int a);
        default double sqrt(int a){
            return Math.sqrt(a);
        }
    }

Formula接口除了抽象方法计算接口方式还定义了默认方法sqrt。实现该接口的类只需要实现抽象方法calculate。默认方法sqrt可以直接使用。当然你也可以直接通过接口创建对象，然后实现接口中的默认方法就可以了。

    public class Main {
        public static void main(String[] args){
            Formula formula = new Formula() {
                @Override
                public double calculate(int a) {
                    return sqrt(a*100);
                }
            };
            System.out.println(formula.calculate(100));
            System.out.println(formula.sqrt(16));
        }
    }

**注意:** 不管是抽象类还是接口，可以通过内部类的方式访问，但是不可以直接定义对象。内部类的实现，可以理解为内部类实现了接口的抽象方法，并返回一个内部类对象，之后我们让接口的引用指向这个对象。

