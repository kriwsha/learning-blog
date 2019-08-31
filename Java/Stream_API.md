# Java Stream API

## 1. Создание потока
```
List<String> words = Arrays.asList(content.split("\\s+"));

Stream<String> stream = words.stream();
Stream<String> stream = words.parallelStream(); 
        // при параллельном потоке обработка происходит в случайном порядке
Stream<String> stream = Stream.of(content.split("\\s+"));
Stream<String> stream = Arrays.stream(content.split("\\s+"));
Stream<String> stream = Stream.empty(); // пустой поток
Stream<Double> stream = Stream.generate(Math::random);
Stream<BigInteger> stream =     
        Stream.itarate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
            // бесконечная последовательность 0, 1, 2, ...
Stream<String> stream = Pattern.compile("\\s+").splitAsStream(content);
Stream<String> stream = Files.lines(new File("file/path").toPath());
```

## 2. Методы filter(), map(), flatMap()
```
Stream<String> stream = words.stream().filter(w -> w.length() > 3);
Stream<String> stream = words.stream().map(String::toLowerCase);
        // Применить к каждому элементу функцию
Stream<String> stream = words.stream().map(s -> s.substring(0, 1));
Stream<String> stream = words.stream().flatMap(this::leters);
        // letters - пользовательска функция, которая возвращает поток букв 
        // для каждого элемента, н-р,
        //      private Stream<String> letters(String word) {
        //          return Stream.of(word.toCharArray()).map(String::new);   
        //      }
        // flatMap возвращает один поток вместо списка
```

## 3. Извлечение подпотоков и сцепление потоков
```
Stream<Double> stream = Stream.generate(Math::random).limit(100);
        // получаем 100 первых элементов
Stream<String> stream = Stream.of(content.split("\\s+)).skip(1);
        // пропускаем первый элемент
Stream<String> stream = Stream.concat(letters("Hello"), letters("World));
        // объединяем потоки
```

## 4. Другие потоки
```
Stream<String> stream = Stream.of("mer", "dir", "em", "mer").distinct();
            // удаяляем "mer"
Stream<String> stream = words.stream()
        .sorted(Comparator.comparing(String::length).reversed());
            // сортировка по компаратору
Stream<String> stream = Stream.iterate(1.0, p -> p*2)
        .peek(e -> System.out.println(e))
        .limit(20);
            // peek - для промежуточных операций, 
            // не изменяющих поток (н-р, отладки)
```

## 5. Методы сведения
```
Optional<String> largest = words.stream().max(String::compareToIgnoreCase);
        // возвращает максимальное значение созгласно функции компаратора
        // Optional<?> largest - объект-обертка, который может хранить
        // внутри себя реальный объект, получаемый из стрима
System.out.println("largest: " + largest.orElse("no"));
        // выводим самое длинное слово или "no", если такое не найдено
Optional<String> largest = words.stream()
        .filter(s -> s.startsWith("Q"))
        .findFirst();
        // возвращает первое слово, начинающееся на Q либо ничего
Optional<String> largest = words.parallelStream()
        .filter(s -> s.startsWith("Q"))
        .findAny();
        // возвращает первое слово, начинающееся на Q либо ничего
boolean bool = words.parallelStream().anyMatch(s -> s.startsWith("Q"));
        // проверяет, есть ли совпадения с условием
```

## 6. Способы использования groupingBy
```
// Группировка списка рабочих по должности (деление на саписки)
Map<String, List<Worker>> map = workers.stream()
        .collect(Colletors.groupingBy(Worker::getPosition));

// Группировка списка рабочих по должности (деление на множества)
Map<String, Set<Worker>> map = workers.stream()
        .collect(Colletors.groupingBy(Worker::getPosition, 
                Collectors.toSet()));

// Подсчет количества рабочих, занимающих конкретные должности
Map<String, Long> map = workers.stream()
        .collect(Collectors.groupingBy(Worker::getPosition,
                Collectors.counting()));

// Группировка списка рабочих по их должности (интересуют только имена)
Map<String, Set<String>> map = workers.stream()
        .collect(Collectors.groupingBy(Worker::getPosition,
                Collectors.mapping(Worker::getName,
                        COllectors.toSet())));

// Расчет средней з/п для данной должности
Map<String, Double> map = workers.stream()
        .collect(Collectors.groupingBy(Worker::getPosition,
                Collectors.abaragingInt(Worker::getSalary)));

// Группировка по должности, имена одной строкой
Map<String, String> map = workers.stream()
        .collect(Collectors.groupingBy(Worker::getPosition,
                Collectores.mapping(Worker::getName,
                    Collectors.joining(", ", "{", "}"))));

// Группировка списка по должности и возрасту
Map<String, Map<Integer, List<Worker>>> collect = workers.stream()
        .collect(Collectors.groupingBy(Worker::getPosition,
                Collectors.groupingBy(Worker::getAge)));
```