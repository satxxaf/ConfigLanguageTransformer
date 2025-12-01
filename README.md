# Конвертер учебного конфигурационного языка в JSON

## Содержание:
- [О домашней работе](#о-домашней-работе)
- [Условия Варианта №1](#условия-варианта-№1)

## О домашней работе
Домашняя работа (ДР) выполняется дистанционно. Результаты работы над ДР сохраняются в публично доступном git-репозитории. Ссылка на публично доступный git-репозиторий 
с результатами выполнения ДР загружается в СДО. Этапы работы над ДР должны быть отражены в истории коммитов с детальными сообщениями. Студент самостоятельно выбирает язык реализации и
специализированный инструмент синтаксического разбора. Решения без использования специализированных инструментов синтаксического разбора не засчитываются.
Документация по ДР оформляется в виде readme.md, который содержит:
1. Общее описание.
2. Описание всех функций и настроек.
3. Описание команд для сборки проекта и запуска тестов.
4. Примеры использования
   
## Условия Варианта №1
Разработать инструмент командной строки для учебного конфигурационного языка, синтаксис которого приведен далее. Этот инструмент преобразует текст из
входного формата в выходной. Синтаксические ошибки выявляются с выдачей сообщений.
Входной текст на учебном конфигурационном языке принимается из файла, путь к которому задан ключом командной строки. Выходной текст на
языке json попадает в файл, путь к которому задан ключом командной строки.

Числа:
- 0[xX][0-9a-fA-F]+

Массивы:
- #( значение значение значение ... )

Имена:
- [a-zA-Z][a-zA-Z0-9]*

Значения:
- Числа.
- Массивы.

Объявление константы на этапе трансляции:
- global имя = значение

Вычисление константы на этапе трансляции:
- ?[имя]

Результатом вычисления константного выражения является значение. Все конструкции учебного конфигурационного языка (с учетом их возможной вложенности) должны быть покрыты тестами.
Необходимо показать 2 примера описания конфигураций из разных предметных областей.

# Решение
## Общее описание
Данный проект представляет собой инструмент командной строки для преобразования текстовых файлов, написанных на учебном
конфигурационном языке, в стандартный формат JSON. Программа выполняет полный цикл обработки языка: лексический анализ, синтаксический разбор, 
построение абстрактного синтаксического дерева (AST) и генерацию JSON-выхода.

## Основные возможности
- Поддержка шестнадцатеричных чисел
- Работа с массивами и объектами
- Объявление и использование констант на этапе трансляции
- Детальная обработка ошибок с указанием позиции в исходном коде
- Полное покрытие тестами всех конструкций языка
  
## Описание всех функций и настроек
### Архитектура проекта
**Проект реализован на C++ и состоит из следующих основных компонентов:**

Валидация параметров:
1. Лексический анализатор (Lexer)
- Назначение: Разбивает входной текст на токены
- Поддерживаемые токены:
  - NUMBER: шестнадцатеричные числа (0[xX][0-9a-fA-F]+)
  - STRING: строки в двойных кавычках
  - IDENTIFIER: идентификаторы ([a-zA-Z][a-zA-Z0-9]*)
  - GLOBAL: ключевое слово для объявления констант
  - Символы: {, }, [, ], (, ), #, =, ?
3. Синтаксический анализатор (Parser)
- Назначение: Строит абстрактное синтаксическое дерево из токенов
- Поддерживаемые узлы AST:
  - NumberNode: представление числовых значений
  - StringNode: представление строковых значений
  - BoolNode: представление логических значений
  - ArrayNode: представление массивов
  - ObjectNode: представление объектов
4. Система констант
  - Объявление констант: global имя = значение
  - Использование констант: ?[имя]
  - Особенности: Константы вычисляются на этапе трансляции

### Поддерживаемые конструкции языка
Числа
```
cpp
port = 0x1A
timeout = 0xFF
```
Массивы
```
cpp
ports = #( 0x01 0x02 0x03 )
hosts = #( "server1" "server2" "server3" )
Объекты
cpp
server = {
    port = 0x50
    enabled = true
    timeout = 0x1E
}
```
Константы
```
cpp
global MAX_SIZE = 0x100
global DEFAULT_PORT = 0x50

buffer_size = ?[MAX_SIZE]
port = ?[DEFAULT_PORT]
```

## Описание команд для сборки проекта и запуска тестов
**Требования:**

Компилятор C++

Операционная система: Windows/Linux/macOS

### Сборка проекта
Компиляция с g++
```
g++ -std=c++11 -o ConfigLanguageTransformer main.cpp
```

Компиляция с clang++
```
clang++ -std=c++11 -o ConfigLanguageTransformer main.cpp
```

Компиляция с оптимизацией
```
g++ -std=c++11 -O2 -o ConfigLanguageTransformer main.cpp
```

### Запуск тестов
#### Запуск всех тестов
```
./ConfigLanguageTransformer --test
```

Скрин результата:
<img width="1121" height="870" alt="image" src="https://github.com/user-attachments/assets/e752c948-0481-49f1-b27d-747205859235" />

#### Преобразование конфигурационного файла в JSON файл
```
./ConfigLanguageTransformer --input input.txt --output output.json
```
То что выведет:

Успешно преобразованный "путь к файлу txt" к "путь для сохранения json"



### Код работы
```
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <map>
#include <memory>
#include <sstream>
#include <iomanip>
#include <cctype>

using namespace std;

enum class TokenType {
    NUMBER, STRING, IDENTIFIER, LBRACE, RBRACE,
    LBRACKET, RBRACKET, LPAREN, RPAREN, HASH, EQUALS, QUESTION,
    GLOBAL, EOF_TOKEN, INVALID
};

struct Token {
    TokenType type;
    string value;
    int line;
    int column;

    Token(TokenType t, const string& v = "", int l = 0, int c = 0)
        : type(t), value(v), line(l), column(c) {
    }
};

class ASTNode {
public:
    virtual ~ASTNode() = default;
    virtual string toJSON(int indent = 0) const = 0;
};

class NumberNode : public ASTNode {
    long long value;
public:
    NumberNode(long long v) : value(v) {}
    string toJSON(int indent = 0) const override {
        return to_string(value);
    }
    long long getValue() const { return value; }
};

class StringNode : public ASTNode {
    string value;
public:
    StringNode(const string& v) : value(v) {}
    string toJSON(int indent = 0) const override {
        return "\"" + value + "\"";
    }
};

class BoolNode : public ASTNode {
    bool value;
public:
    BoolNode(bool v) : value(v) {}
    string toJSON(int indent = 0) const override {
        return value ? "true" : "false";
    }
};

class ArrayNode : public ASTNode {
    vector<shared_ptr<ASTNode>> elements;
public:
    void addElement(shared_ptr<ASTNode> element) {
        elements.push_back(element);
    }
    string toJSON(int indent = 0) const override {
        string result = "[";
        for (size_t i = 0; i < elements.size(); ++i) {
            if (i > 0) result += ", ";
            result += elements[i]->toJSON();
        }
        result += "]";
        return result;
    }
};

class ObjectNode : public ASTNode {
    map<string, shared_ptr<ASTNode>> properties;
public:
    void addProperty(const string& key, shared_ptr<ASTNode> value) {
        properties[key] = value;
    }
    string toJSON(int indent = 0) const override {
        if (properties.empty()) return "{}";

        string result = "{\n";
        string indentStr(indent + 2, ' ');
        bool first = true;
        for (const auto& prop : properties) {
            if (!first) result += ",\n";
            result += indentStr + "\"" + prop.first + "\": " + prop.second->toJSON(indent + 2);
            first = false;
        }
        result += "\n" + string(indent, ' ') + "}";
        return result;
    }
};

class Lexer {
    string input;
    size_t position;
    int line;
    int column;

    char peek() const {
        return position < input.length() ? input[position] : '\0';
    }

    char advance() {
        char c = peek();
        if (c != '\0') {
            position++;
            if (c == '\n') {
                line++;
                column = 1;
            }
            else {
                column++;
            }
        }
        return c;
    }

    void skipWhitespace() {
        while (position < input.length() && isspace(peek())) {
            advance();
        }
    }

    bool isHexDigit(char c) {
        return isdigit(c) || (c >= 'a' && c <= 'f') || (c >= 'A' && c <= 'F');
    }

public:
    Lexer(const string& text) : input(text), position(0), line(1), column(1) {}

    Token nextToken() {
        skipWhitespace();

        if (position >= input.length()) {
            return Token(TokenType::EOF_TOKEN, "", line, column);
        }

        char current = peek();
        int startLine = line;
        int startColumn = column;

        if (current == '0' && position + 1 < input.length() &&
            (input[position + 1] == 'x' || input[position + 1] == 'X')) {
            advance();
            advance();
            string number;
            while (position < input.length() && isHexDigit(peek())) {
                number += advance();
            }
            return Token(TokenType::NUMBER, number, startLine, startColumn);
        }

        if (isalpha(current)) {
            string identifier;
            while (position < input.length() && (isalnum(peek()) || peek() == '_')) {
                identifier += advance();
            }
            if (identifier == "global") return Token(TokenType::GLOBAL, identifier, startLine, startColumn);
            if (identifier == "true" || identifier == "false") return Token(TokenType::STRING, identifier, startLine, startColumn);
            return Token(TokenType::IDENTIFIER, identifier, startLine, startColumn);
        }

        if (current == '"') {
            advance();
            string str;
            while (position < input.length() && peek() != '"' && peek() != '\0') {
                str += advance();
            }
            if (peek() == '"') advance();
            return Token(TokenType::STRING, str, startLine, startColumn);
        }

        // одиночные символы
        switch (current) {
        case '{': advance(); return Token(TokenType::LBRACE, "{", startLine, startColumn);
        case '}': advance(); return Token(TokenType::RBRACE, "}", startLine, startColumn);
        case '[': advance(); return Token(TokenType::LBRACKET, "[", startLine, startColumn);
        case ']': advance(); return Token(TokenType::RBRACKET, "]", startLine, startColumn);
        case '(': advance(); return Token(TokenType::LPAREN, "(", startLine, startColumn);
        case ')': advance(); return Token(TokenType::RPAREN, ")", startLine, startColumn);
        case '#': advance(); return Token(TokenType::HASH, "#", startLine, startColumn);
        case '=': advance(); return Token(TokenType::EQUALS, "=", startLine, startColumn);
        case '?': advance(); return Token(TokenType::QUESTION, "?", startLine, startColumn);
        }

        // неизвестный символ
        string invalidChar(1, current);
        advance();
        return Token(TokenType::INVALID, invalidChar, startLine, startColumn);
    }
};

class Parser {
    Lexer& lexer;
    Token currentToken;
    map<string, shared_ptr<ASTNode>> constants;

    void eat(TokenType expected) {
        if (currentToken.type == expected) {
            currentToken = lexer.nextToken();
        }
        else {
            throw runtime_error("Синтаксическая ошибка в строке " + to_string(currentToken.line) +
                ", column " + to_string(currentToken.column) +
                ": expected " + tokenTypeToString(expected) +
                ", got " + tokenTypeToString(currentToken.type));
        }
    }

    string tokenTypeToString(TokenType type) {
        switch (type) {
        case TokenType::NUMBER: return "NUMBER";
        case TokenType::STRING: return "STRING";
        case TokenType::IDENTIFIER: return "IDENTIFIER";
        case TokenType::LBRACE: return "LBRACE";
        case TokenType::RBRACE: return "RBRACE";
        case TokenType::LBRACKET: return "LBRACKET";
        case TokenType::RBRACKET: return "RBRACKET";
        case TokenType::LPAREN: return "LPAREN";
        case TokenType::RPAREN: return "RPAREN";
        case TokenType::HASH: return "HASH";
        case TokenType::EQUALS: return "EQUALS";
        case TokenType::QUESTION: return "QUESTION";
        case TokenType::GLOBAL: return "GLOBAL";
        case TokenType::EOF_TOKEN: return "EOF_TOKEN";
        case TokenType::INVALID: return "INVALID";
        default: return "UNKNOWN";
        }
    }

    shared_ptr<ASTNode> parseValue() {
        if (currentToken.type == TokenType::NUMBER) {
            long long value = stoll(currentToken.value, nullptr, 16);
            auto node = make_shared<NumberNode>(value);
            eat(TokenType::NUMBER);
            return node;
        }
        else if (currentToken.type == TokenType::STRING) {
            if (currentToken.value == "true" || currentToken.value == "false") {
                auto node = make_shared<BoolNode>(currentToken.value == "true");
                eat(TokenType::STRING);
                return node;
            }
            else {
                auto node = make_shared<StringNode>(currentToken.value);
                eat(TokenType::STRING);
                return node;
            }
        }
        else if (currentToken.type == TokenType::HASH) {
            eat(TokenType::HASH);
            eat(TokenType::LPAREN);
            auto array = make_shared<ArrayNode>();
            while (currentToken.type != TokenType::RPAREN && currentToken.type != TokenType::EOF_TOKEN) {
                array->addElement(parseValue());
            }
            eat(TokenType::RPAREN);
            return array;
        }
        else if (currentToken.type == TokenType::QUESTION) {
            eat(TokenType::QUESTION);
            eat(TokenType::LBRACKET);
            string constantName = currentToken.value;
            eat(TokenType::IDENTIFIER);
            eat(TokenType::RBRACKET);

            if (constants.find(constantName) == constants.end()) {
                throw runtime_error("Неизвестная константа: " + constantName);
            }
            return constants[constantName];
        }
        else if (currentToken.type == TokenType::LBRACE) {
            return parseObject();
        }

        throw runtime_error("Неожиданный токен в значении: " + currentToken.value + " в строке " + to_string(currentToken.line));
    }

    shared_ptr<ObjectNode> parseObject() {
        auto obj = make_shared<ObjectNode>();
        eat(TokenType::LBRACE);

        while (currentToken.type != TokenType::RBRACE && currentToken.type != TokenType::EOF_TOKEN) {
            if (currentToken.type == TokenType::IDENTIFIER) {
                string key = currentToken.value;
                eat(TokenType::IDENTIFIER);
                eat(TokenType::EQUALS);
                auto value = parseValue();
                obj->addProperty(key, value);
            }
            else {
                throw runtime_error("Ожидаемый идентификатор в объекте");
            }
        }

        eat(TokenType::RBRACE);
        return obj;
    }

public:
    Parser(Lexer& l) : lexer(l), currentToken(l.nextToken()) {}

    shared_ptr<ASTNode> parse() {
        auto root = make_shared<ObjectNode>();

        while (currentToken.type != TokenType::EOF_TOKEN) {
            if (currentToken.type == TokenType::GLOBAL) {
                eat(TokenType::GLOBAL);
                string name = currentToken.value;
                eat(TokenType::IDENTIFIER);
                eat(TokenType::EQUALS);
                auto value = parseValue();
                constants[name] = value;
            }
            else if (currentToken.type == TokenType::IDENTIFIER) {
                string key = currentToken.value;
                eat(TokenType::IDENTIFIER);
                eat(TokenType::EQUALS);
                auto value = parseValue();
                root->addProperty(key, value);
            }
            else if (currentToken.type == TokenType::LBRACE) {
                auto obj = parseObject();
                root->addProperty("unnamed", obj);
            }
            else {
                throw runtime_error("Неожиданный токен: " + currentToken.value + " в строке " + to_string(currentToken.line));
            }
        }

        return root;
    }
};

void runTests() {
    cout << "Выполнение тестов...\n";

    // Тест 1: Простое число
    {
        Lexer lexer("port = 0x1A");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 1 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 1 не пройден: " << e.what() << endl;
        }
    }

    // Тест 2: Массив
    {
        Lexer lexer("ports = #( 0x01 0x02 0x03 )");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 2 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 2 не пройден: " << e.what() << endl;
        }
    }

    // Тест 3: Константы
    {
        Lexer lexer("global MAX_SIZE = 0x100\nsize = ?[MAX_SIZE]");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 3 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 3 не пройден: " << e.what() << endl;
        }
    }

    // Тест 4: Объект
    {
        Lexer lexer("config = { timeout = 0x1E enabled = true }");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 4 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 4 не пройден: " << e.what() << endl;
        }
    }

    // Тест 5: Комплексный пример
    {
        Lexer lexer("global PORT = 0x50\nserver = { port = ?[PORT] hosts = #( \"host1\" \"host2\" ) }");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 5 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 5 не пройден: " << e.what() << endl;
        }
    }

    // Тест 6: Вложенные объекты
    {
        Lexer lexer("app = { database = { host = \"localhost\" port = 0x2276 } }");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 6 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 6 не пройден: " << e.what() << endl;
        }
    }

    // Тест 7: Смешанные типы данных
    {
        Lexer lexer("settings = { numbers = #( 0x01 0x02 ) strings = #( \"a\" \"b\" ) flag = true }");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 7 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 7 не пройден: " << e.what() << endl;
        }
    }

    // Тест 8: Множественные константы
    {
        Lexer lexer("global WIDTH = 0x500\nglobal HEIGHT = 0x300\ndimensions = { width = ?[WIDTH] height = ?[HEIGHT] }");
        Parser parser(lexer);
        try {
            auto result = parser.parse();
            cout << "Тест 8 пройден: " << result->toJSON() << endl;
        }
        catch (const exception& e) {
            cout << "Тест 8 не пройден: " << e.what() << endl;
        }
    }

    cout << "Тесты завершены.\n";
}

int main(int argc, char* argv[]) {
    setlocale(LC_ALL, "RU");
    if (argc == 2 && string(argv[1]) == "--test") {
        runTests();
        return 0;
    }

    if (argc != 5) {
        cerr << "Usage: " << argv[0] << " --input <input_file> --output <output_file>\n";
        cerr << "Or: " << argv[0] << " --test\n";
        return 1;
    }

    string inputFile, outputFile;
    for (int i = 1; i < argc; i++) {
        if (string(argv[i]) == "--input") {
            inputFile = argv[++i];
        }
        else if (string(argv[i]) == "--output") {
            outputFile = argv[++i];
        }
    }

    if (inputFile.empty() || outputFile.empty()) {
        cerr << "Должны быть указаны как входные, так и выходные файлы\n";
        return 1;
    }

    try {
        ifstream inFile(inputFile);
        if (!inFile) {
            throw runtime_error("Не удается открыть входной файл: " + inputFile);
        }

        stringstream buffer;
        buffer << inFile.rdbuf();
        string inputText = buffer.str();
        inFile.close();

        Lexer lexer(inputText);
        Parser parser(lexer);
        auto ast = parser.parse();

        ofstream outFile(outputFile);
        if (!outFile) {
            throw runtime_error("Не удается открыть выходной файл: " + outputFile);
        }

        outFile << ast->toJSON();
        outFile.close();
        cout << "Успешно преобразованный " << inputFile << " к " << outputFile << endl;

    }
    catch (const exception& e) {
        cerr << "Ошибка: " << e.what() << endl;
        return 1;
    }

    return 0;
}
```
