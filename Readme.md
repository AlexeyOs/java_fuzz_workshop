# Fuzzing pdfbox

## Подготовка окружения

Собираем docker образ:

``` #bash
git clone https://github.com/saladosss/java_fuzz_workshop.git && cd java_fuzz_workshop
docker build --tag=pdfbox_workshop_img .
```

Запускаем контейнер:

``` #bash
docker run -it -v "$(pwd)/artifacts:/home/fuzz/artifacts:ro" --name=pdfbox_fuzz pdfbox_workshop_img
```

Собираем проект:

``` #bash
cd /home/fuzz/pdfbox
mvn clean install -DskipTests
```

## Фаззинг java кода с помощью jazzer

Cобираем обёртку для pdfbox:

``` #bash
cd /home/fuzz/pdfbox
mkdir fuzz && cd fuzz
cp -r /home/fuzz/artifacts/* .
javac -cp ../pdfbox/target/classes/ ./PDFStreamParserFuzzer.java
```

Запускаем фаззинг PDF парсера:

``` #bash
jazzer --cp=.:../pdfbox/target/classes/:../io/target/classes/:/home/fuzz/log4j/log4j-api-2.24.3.jar:/home/fuzz/log4j/commons-logging-1.2/commons-logging-1.2.jar --target_class=PDFStreamParserFuzzer -dict=pdf.dict -close_fd_mask=3 -- corpus
```

## Сбор покрытия

Прогоняем наработанный корпус:

``` #bash
cd /home/fuzz/pdfbox/fuzz
jazzer --cp=.:../pdfbox/target/classes/:../io/target/classes/:/home/fuzz/log4j/log4j-api-2.24.3.jar::/home/fuzz/log4j/commons-logging-1.2/commons-logging-1.2.jar --target_class=PDFStreamParserFuzzer -close_fd_mask=3 --coverage_dump=coverage.exec -runs=1 -- corpus
```

Генерируем html отчёт с помощью jacococli:

``` #bash
java -jar /home/fuzz/jacoco/lib/jacococli.jar report coverage.exec --classfiles ../pdfbox/target/pdfbox-3.0.4.jar --html report --sourcefiles ../pdfbox/src/main/java/
```

Копируем папку с html отчётом на хост:

``` #bash
docker cp pdfbox_fuzz:/home/fuzz/pdfbox/fuzz/report .
```
