https://devdocs.magento.com/guides/v2.2/extension-dev-guide/build/di-xml-file.html

**Обзор**

Файл di.xml настраивает, какие `зависимости` вводятся диспетчером объектов(object manager). Вы так же можете указать чувствительные настройки конфигурации (Файл di.xml настраивает, какие `зависимости` вводятся диспетчером объектов(object manager). Вы так же можете указать чувствительные настройки конфигурации (sensitive configuration settings) используя di.xml

**Зоны и точки входа приложения.**

Каждый модуль может иметь глобальные и зависимый от зоны(area-specific) `di.xml`. Magento читает все di.xml файлы конфигурации обьявленные в системе и сливает их все вместе дбавляя все узлы. _В общем из кучи di.xml собирается один большой для системы_

Как общее правило зоно зависимые di.xml файлы должен конфигурировать зависимости для уровня представления(presentation layer), а глобальный для модуля di.xml должен сконфигурировать все остальные зависимости.
 

Magento загружает конфигурацию в следующих стадия:

1. Initial (app/etc/di.xml)
2. Global (<moduleDir>/etc/di.xml)
3. Area-specific (<moduleDir>/etc/<area>/di.xml)

Во время бустрапа(bootstrapping) каждая точка входа приложения загружает соответствующие файлы di.xml для запрашиваемой области.

**Примеры:**

* In index.php, the \Magento\Framework\App\Http class loads the area based on the front-name provided in the URL .

* In static.php, the \Magento\Framework\App\StaticResource class also loads the area based on the URL in the request.

* In cron.php, the \Magento\Framework\App\Cron class always loads the crontab area.

**Типы конфигурации**

Тип конфигурации описывает образ жизни обьекта и как его создать.

Вы можете сконфигурировать этот тип в своем конфигурационном узле(node) di.xml следующими путями:

```
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <virtualType name="moduleConfig" type="Magento\Core\Model\Config">
        <arguments>
            <argument name="type" xsi:type="string">system</argument>
        </arguments>
    </virtualType>
    <type name="Magento\Core\Model\App">
        <arguments>
            <argument name="config" xsi:type="object">moduleConfig</argument>
        </arguments>
    </type>
</config>
```

В предыдущем примере объявляются следующие типы:

* `moduleConfig`: Виртуальный тип, который расширяет тип: `Magento\Core\Model\Config`
*  `Magento\Core\Model\App`: Все экземпляры этого типа получают экземпляр `moduleConfig` как зависимость


**Виртуальные типа**

Виртуальные типы разрешают вам изменить аргументы определенной иньектируемой зависиости и изменять поведение определенного класса.
Это позволяет использовать настраиваемый класс, не затрагивая другие классы, которые имеют зависимость от оригинала

В примере создается виртуальный тип для `Magento \ Core \ Model \ Config` и задает `system` как аргумент конструктора для `type`.

**Аргументы конструктора**

Вы можете сконфигурировать аргументы конструктора класса в вашем `di.xml` в argument node(в узле аргумента). Диспетчер объектов(object manager) внедряет эти аргументы в класс во время его создания.Имя аргумента сконфигурированного в XML файле должно соответствовать имени параметра в конструкторе в конфигурируемом классе.

В следующем пример создаются экземпляры `Magento\Core\Model\Session` c аргументами конструктора `$sessionName` задающие значения `adminhtml`
```
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Core\Model\Session">
        <arguments>
            <argument name="sessionName" xsi:type="string">adminhtml</argument>
        </arguments>
    </type>
</config>
```

**Типы аргументов**

`object`

**Node Formats:**

`<argument xsi:type="object">{typeName}</argument>`

`<argument xsi:type="object" shared="{shared}">{typeName}</argument>`

Создают экземпляр типа `typeName` и передает его в качестве аргумента. Вы можете передать любое имя класса, имя интерфейса или виртуальный тип как `typeName`.

Настройка `shared` свойства определяет стиль жизни созданного экземпляра. Смотри [object lifestyle configuration](https://devdocs.magento.com/guides/v2.2/extension-dev-guide/build/di-xml-file.html#object-lifestyle-configuration)

===================================================


`string`

Node Formats:

`<argument xsi:type="string">{strValue}</argument>`

`<argument xsi:type="string" translate="true">{strValue}</argument>`

Magento интерпретирует любое значение для этого узла аргумента как строку.

=====================================================

`boolean`

**Node Format:**

`<argument xsi:type="boolean">{boolValue}</argument>`

Magento преобразует любое значение для этого аргумента в булевское значение. См. Таблицу ниже:

|Input Type | Data | Boolean Value|
|---------- | ---- | --------------
Boolean|true|	true
Boolean|false|	false
String |“true”*|	true
String |“false”*|	false
String |“1”|	true
String	|“0”|	false
Integer	|1|	true
Integer	|0|	false

* Эти строковые литералы чувствительны к регистру

`number`

**Node Format:**

`<argument xsi:type="number">{numericValue}</argument>`

Допустимые значения для этого типа включают в себя:  integers, floats, or [numeric strings](http://us3.php.net/is_numeric).

`init_parameter`

Что-то


**Argument Examples:**

```
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Example\Type">
        <arguments>
            <!-- Pass simple string -->
            <argument name="stringParam" xsi:type="string">someStringValue</argument>
            <!-- Pass instance of Magento\Some\Type -->
            <argument name="instanceParam" xsi:type="object">Magento\Some\Type</argument>
            <!-- Pass true -->
            <argument name="boolParam" xsi:type="boolean">1</argument>
            <!-- Pass 1 -->
            <argument name="intParam" xsi:type="number">1</argument>
            <!-- Pass application init argument, named by constant value -->
            <argument name="globalInitParam" xsi:type="init_parameter">Magento\Some\Class::SOME_CONSTANT</argument>
            <!-- Pass constant value -->
            <argument name="constantParam" xsi:type="const">Magento\Some\Class::SOME_CONSTANT</argument>
            <!-- Pass null value -->
            <argument name="optionalParam" xsi:type="null"/>
            <!-- Pass array -->
            <argument name="arrayParam" xsi:type="array">
                <!-- First element is value of constant -->
                <item name="firstElem" xsi:type="const">Magento\Some\Class::SOME_CONSTANT</item>
                <!-- Second element is null -->
                <item name="secondElem" xsi:type="null"/>
                <!-- Third element is a subarray -->
                <item name="thirdElem" xsi:type="array">
                    <!-- Subarray contains scalar value -->
                    <item name="scalarValue" xsi:type="string">ScalarValue</item>
                    <!-- and application init argument -->
                    <item name="globalArgument " xsi:type="init_parameter">Magento\Some\Class::SOME_CONSTANT</item>
                </item>
            </argument>
        </arguments>
    </type>
</config>
```