# jvm-understanding

До вызова метода main() JVM должна загрузить класс JvmComprehension.

**ClassLoader'ы**

Загрузка происходит по цепочке загрузчиков классов:
- Bootstrap ClassLoader - загружает базовые классы Java (java.lang.Object, String, Integer, System и др.).
- Platform (Extension) ClassLoader - загружает классы платформы Java.
- Application ClassLoader - загружает класс JvmComprehension из classpath приложения.

После загрузки выполняются этапы:
- Loading (загрузка байткода)
- Linking
  - Verification (проверка байткода)
  - Preparation (выделение памяти под статические поля)
  - Resolution (разрешение ссылок)
  - Initialization (инициализация класса)

Информация о классе помещается в Metaspace.

**Комменты в коде**

public class JvmComprehension {

    public static void main(String[] args) {
        int i = 1;                      // 1 В локальных переменных текущего фрейма выделяется место под int, i присваивается значение, записывается в стек.
        Object o = new Object();        // В куче выделяется память под экземпляр Object, вызывается конструктор, ссылка на объект записывается в стек.
        Integer ii = 2;                 // 3 Объект уже в Integer пулле. Поскольку значение в диапазоне -128:127, используется уже существующий объект Integer(2) из кэша. В стек записывается переменная ii с ссылкой на этот уже существующий объект.
        printAll(o, i, ii);             // 4 Создается новый стековый фрейм метода printAll. Во фрейм копируются параметры: o, i, ii.
        System.out.println("finished"); // 7 Во фрейме main вызывается метод println. "finished" уже есть в Srting пулле JVM помещает его в String Pool при создании класса
    }
    
    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5 Значение 700 вне диапазона -128:127, поэтому создается новый объект в хипе, ссылка на этот объект сохраняется в uselessVar в стеке.
        System.out.println(o.toString() + i + ii);  // 6 Вызывается toString(), возвращается строка, создаются временные объекты (StringBuilder (последовательно вызываются append(String), append(int), append(Integer)), String), затем выполняется println(). После выхода из метода printAll() его фрейм удаляется с его локальными переменными (o, i, ii, uselessVar). Объект в хипе Integer(700) больше не имеет ссылок. Он становится кандидатом на сборку мусора.
    }
}
