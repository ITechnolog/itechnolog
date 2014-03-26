#Pericles 2.0

Программный модуль учета и систематизации платежей, поступивших от плательщиков в адрес поставщиков услуг</p>
<b>Характеристика и функциональные возможности  модуля Pericles:</b>

Модуль Pericles  обеспечивает информационно-технологическое  взаимодействие между участниками расчетов, в том числе Поставщиком и Плательщиком, при совершение действий, направленных на осуществление Переводов, для  последующего исполнения денежных обязательств Плательщика перед Поставщиком;

 Модуль Pericles обеспечивает достоверность, целостность и неизменность Информации о Переводах , передаваемой посредством данного Модуля;
 
Модуль Pericles обеспечивает посредством функции <b>Повторные платежи</b> использование  реквизитов первого платежа. Получив реквизиты для первого платежа, Плательщик может пользоваться ими и дальше для регулярного осуществления Переводов в любое время, без повторного посещения сайта и заполнения форм. Для этого достаточно воспользоваться сервисом и указать требуемые данные;

Модуль Pericles обеспечивает <b>Безопасность</b>. Все операции проводятся в защищенной инфраструктуре, надежность которой подтверждена сертификатом PCI DSS Level 1.

<b>Основные принципы проведения платежа</b>

Pericles проводит платежи в 2 этапа:
<ol>
<li>проверка статуса Плательщика и 2) проведение платежа.</li>
<ul>
<li>При проверке статуса платежное приложение проекта должно проверить наличие в базе проекта Плательщика с указанными реквизитами.</li>
<li>При проведении транзакции платежное приложение проекта должно произвести пополнение баланса пользователя и обеспечить отображение информации об осуществленном Переводе. При необходимости проект имеет возможность отменять транзакции.</li>
<li>Общение с Pericles происходит в режиме запрос-ответ. Тип запроса передается системой в переменной command — строка, принимающая значения check, pay и cancel.</li>
<li>База данных проекта не должна содержать две успешно проведенные транзакции с одним и тем же id. Если система присылает запрос повторно с уже существующим в базе id, то платежное приложение проекта должно вернуть результат обработки предыдущего запроса.</li>
</ul>
</ol>
Требования к платежному приложению

Платежное приложение проекта должно:

1. Принимать HTTP или HTTPS запросы со следующих IP адресов: 94.103.26.176/29, 159.255.220.240/28, 185.30.20.16/29 и 185.30.21.16/29.
2. Обрабатывать параметры, передаваемые системой методом GET.
3. Формировать ответ в формате XML в кодировке UTF-8.
4. Обмениваться информацией в режиме запрос-ответ, при этом скорость ответа не должна превышать 60 секунд. В противном случае система разрывает соединение по тайм-ауту.

##Начало работы

Как только указанный в настоящем документе протокол в приложении реализован, пользователи имеют возможность совершать Переводы. После того, как была произведена оплата, на ваш сервер будет отправлено уведомление. Если по какой-либо причине Перевод должен быть отменен, на ваш сервер также будет отправлено уведомление. Сервер, в свою очередь, должен обработать эти уведомления и сформировать ответ.

Формат запроса

Pericles присылает запрос в HTTP или HTTPS.

Формируется URL следующего вида (пример):

<pre>https://example.project.com/?command=pay&id=14332453&v1=User&v2=&v3=&sum=902.48&date=2013-03-2518:48:22&md5=59b5f8cbc147e180df38200348fa962a</pre>

Каждый Перевод в Pericles имеет уникальный идентификационный номер, который передается в параметре id. По этому идентификатору производится дальнейшая сверка взаиморасчетов.

В запросе на проведение Перевода  Pericles передает дату платежа в параметре date — дата в формате ГГГГ-ММ-ДД ЧЧ:ММ:СС. Эту дату необходимо использовать для проведения сверок и бухгалтерских взаиморасчетов.

Sum — сумма денежных средств - дробное число с точностью до сотых, в качестве разделителя используется "." (точка). 

Формат ответа

Платежное приложение должно возвращать ответ в форме XML в кодировке UTF-8 следующего вида:
<pre>
&lt;?xml version="1.0" encoding="windows-1251"?&gt;
&lt;response&gt;
    &lt;id&gt;1234567&lt;/id&gt;
    &lt;id_shop&gt;9876543&lt;/id_shop&gt;
    &lt;result&gt;0&lt;/result&gt;
    &lt;comment&gt;Success&lt;/comment&gt;
&lt;/response&gt;
</pre>

•<b>id</b> — Уникальный id операции в Pericles;

•<b>id_shop</b> — id транзакции в системе проекта (обязательный параметр);

•<b>result</b> — код завершения;

•<b>comment</b> — описание результата.


<b>Список кодов завершения</b>

<table border="1" cellspacing="0" cellpadding="0" > <thead> 
<tr>
<td><p>Код</p></td>
<td><p>Описание</p></td>
<td><p>Фатальность</p></td>
</tr>

</thead>

<tbody>
<tr>
<td><p>0</p></td>
<td><p>ОK</p></td>
<td> </td>
</tr>

<tr>
<td><p>1</p></td>
<td><p>Временная ошибка, повторите запрос позже</p></td>
<td><p>Нет</p></td>
</tr>

<tr>
<td><p>2</p></td>
<td><p>Неверный идентификатор Плательщика</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>3</p></td>
<td><p>Неверная подпись md5</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>4</p></td>
<td><p>Неверный формат запроса (неверная сумма, неполный набор параметров)</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>5</p></td>
<td><p>Другая ошибка (желательно описать ее в <i>comment</i>)</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>7</p></td>
<td><p>Прием Перевода от данного пользователя запрещен по техническим причинам</p></td>
<td><p>Да</p></td>
</tr>

</tbody>
</table>

###Запросы
Ваш сервер должен отвечать на следующие запросы:

•Запрос на существование Плательщика (<b>check</b>);

•Запрос на осуществление Перевода (<b>pay</b>);

•Запрос на отмену Перевода (<b>cancel</b>).

###Структура базы данных

Примеры, приведенные ниже предполагают, что у вас будет SQL база данных, которая хранит информацию в следующей таблице:
<pre>
&ensp;# A table for storing payment receipts from the system.
CREATE TABLE `payments` (
`id` INT(11) NOT NULL AUTO_INCREMENT,
`invoice` INT(11) NOT NULL,
`v1` VARCHAR(255) NOT NULL,
`v2` VARCHAR(255) NULL,
`v3` VARCHAR(100) NULL,
`payment_date` TIMESTAMP NOT NULL,
`sum` FLOAT NOT NULL,
`canceled` TINYINT NOT NULL DEFAULT 0,
`date_canceled` TIMESTAMP NOT NULL,
PRIMARY KEY (`id`) )
DEFAULT CHARACTER SET = utf8
COLLATE = utf8_bin;
</pre>

##Существование  Плательщика
###Сheck

Сервер отправляет запрос для проверки существования Плательщика в проекте на URL платежного приложения, указанного в личном кабинете. Запрос отправляется методом GET и содержит следующие параметры:

<b>Параметры запроса</b>
<table border="1" cellspacing="0" cellpadding="0" > <thead> <tr>
<td><p>Название поля</p></td>
<td><p>Тип</p></td>
<td><p>Описание</p></td>
<td><p>Обязательность</p></td>
<td><p>Пример</p></td>
</tr>

</thead> 
<tbody>
<tr>
<td><p>command</p></td>
<td><p>String</p></td>
<td><p>Признак того, что идет проверка на существование Плательщика</p></td>
<td><p>Да</p></td>
<td><p>check</p></td>
</tr>

<tr>
<td><p>v1</p></td>
<td><p>String(255 символов)</p></td>
<td><p>Параметр идентификации Плательщика</p></td>
<td><p>Нет</p></td>
<td><p>User</p></td>
</tr>

<tr>
<td><p>v2</p></td>
<td><p>String(200 символов)</p></td>
<td><p>Дополнительный параметр идентификации Плательщика</p></td>
<td><p>Нет</p></td>
<td><p>0</p></td>
</tr>

<tr>
<td><p>v3</p></td>
<td><p>String(100 символов)</p></td>
<td><p>Дополнительный параметр идентификации Плательщика</p></td>
<td><p>Нет</p></td>
<td><p>0</p></td>
</tr>

<tr>
<td><p>md5</p></td>
<td><p>String</p></td>
<td><p>Подпись для предотвращения несанкционированного доступа</p></td>
<td><p>Да</p></td>
<td><p>59b5f8cbc147e180df38200348fa962a</p></td>
</tr>

</tbody>
</table>


###Правила формирования подписи md5
Подпись служит для обеспечения безопасности при проверке и проведении платежей. Формируется с помощью алгоритма хэширования md5 от строки, полученной конкатенацией параметров, представленных ниже:

<pre>md5(&lt;command&gt;&lt;v1&gt;&lt;secret_key&gt;)</pre>

<b>secret_key</b> — секретное слово, которое было указано в личном кабинете.

###Пример
Для формирования подписи необходимы следующие параметры:

•command = check;

•v1 = User;

•secret_key = password.

Строка будет иметь следующий вид:
<pre>checkUserpassword</pre>

md5 хэш от данной строки:
<pre>a285084e00a6ff3fc8c3646e3c7c7e64</pre>

###Проверка ip-адреса и md5 подписи

Перед обработкой запроса проект должен проверить его подлинность. Для этого необходимо проверить принадлежность IP-адреса, с которого пришел запрос, указанному диапазону адресов, а также сравнить значение параметра md5 cо значением хэша md5 от строки, полученной согласно принципу, описанному выше. Если значения не совпадают, то запрос не является подлинным и должен быть проигнорирован. Проект должен уведомить систему об ошибке. Как только подлинность уведомления была проверена, ваш сервер должен продолжить обработку.

Для проверки принадлежности IP-адреса указанным подсетям можно воспользоваться библиотекой <a href="https://github.com/mattallty/Utils-IP-Tools">Utils-IP-Tools</a>.

<b>Пример проверки подлинности запроса:</b>
 <pre>&lt;?php
function verifyMD5(&dollar;secretKey, $command, $v1, $md5)
{
        $md5Hash = md5($command.$v1.$secretKey);
        if(strtolower($md5Hash) != strtolower($md5))
                return(false);
                return(true);
}
?></pre>

###Обработка запроса на существование Плательщика в проекте
Проверка существования Плательщика в проекте и проверка, что осуществление Перевода возможно.

<pre> &lt;?php
$v1 = $_GET['v1'];
// check_nickname – функция(на стороне проекта), которая проверяет наличие данного пользователя в базе данных проекта
$check = check_nickname($v1);
// если ник существует
if ($check) { $result = '0';
$comment = 'success';
} // если ник не существует 
else {
        $result = '7';
        $comment = 'fail'; }
echo '<?xml version="1.0" encoding="windows-1251"?>
    <response>
        <result>' . $result . '</result>
        <comment>' . $comment . '</comment>
    </response>
    ';
    // функция check
    function check_nickname() {
    // ваш код
    return true; }
    ?>
 </pre>   
 
###Ответ
После обработки запроса следует сформировать ответ в формате XML и кодировке UTF-8.
<b>Формирование ответа</b>

 <pre>&lt;?php
function generateResponseCheck($iresult, $strDescription)
{
        $responseXML = new SimpleXMLElement('<response></response>');
            $responseXML->addChild('result', $iresult);
            $responseXML->addChild('comment', $strDescription);
            header('Content-Type: text/xml; charset=utf8');
            echo $responseXML->asXML();
            }
            ?></pre>
            
 
<b>Схема ответа</b>
<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    &lt;xsd:element name="response">
        &lt;xsd:complexType>
            &lt;xsd:all>
                &lt;xsd:element name="result" type="xsd:unsignedInt"/>
                &lt;xsd:element name="comment" type="xsd:string" minOccurs="0"/>
                &lt;xsd:element name="specification" minOccurs="0">
                    &lt;xsd:complexType>
                        &lt;xsd:sequence>
                            &lt;xsd:any namespace="##any" processContents="skip" minOccurs="1" maxOccurs="unbounded"/>
                        &lt;/xsd:sequence>
                    &lt;/xsd:complexType>
                &lt;/xsd:element>
            &lt;/xsd:all>
        &lt;/xsd:complexType>
    &lt;/xsd:element>
&lt;/xsd:schema></pre>

###Примеры XML
в случае, если Плательщик существует и осуществление Перевода возможно:
<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;result>0</result>
&lt;/response></pre>

в противном случае:

<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;result>7</result>
    &lt;comment>Account is disabled or not present</comment>
&lt;/response></pre>
•<b>result</b> — код завершения;

•<b>comment</b> — описание результата.

###Дополнительная анти-фрод защита
Проект может передавать Pericles дополнительные параметры, которые могут предоставить больше информации о том или ином пользователе. Эти дополнительные меры помогут нам в оценке степени доверия пользователю и в более точной настройке фрод-фильтров. XML ответ в этом случае будет следующего вида:

<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;result>...</result>
    &lt;comment>...</comment>
    &lt;specification>
        &lt;s1>...</s1>
        &lt;s2>...</s2>
        ...
    &lt;/specification>
&lt;/response></pre>
Все параметры должны передаваться в соответствии с документацией протокола. Сумма передается в параметре типа Float, разделителем является "." (0, 1, 2 знака после точки). Дата передается в параметре типа String в виде ГГГГ-ММ-ДД ЧЧ:ММ:СС.

Допустим, можно использовать параметр s1 для обозначения степени доверия пользователя (пример: пользователь со степенью доверия ‘100+' - полное доверие); параметр s2 — для даты регистрации, s3 — уровень Плательщика в проекте и т.д. Для настройки параметров и задания значений каждого из них обратитесь к менеджеру проекта.

###Доверенный список Плательщиков
Для обеспечения безопасности каждый Перевод в системе проверяется системой анти-фрод защиты. Если проект имеет ряд доверенных пользователей, то можно передать их список аккаунт-менеджеру. В таком случае, если Перевод доверенного плательщика окажется остановленным анти-фрод защитой, то он будет проверен вручную, что позволит наиболее точно определить, насколько безопасной является данная транзакция. Доверенный список должен содержать уникальные идентификаторы Плательщиков (параметр v1).

###Список кодов завершения
<table border="1" cellspacing="0" cellpadding="0" > <thead> 

<tr>
<td><p>Result</p></td>
<td><p>Описание</p></td>
</tr>
<tbody>
<tr>
<td><p>0</p></td>
<td><p>Платеж с указанными идентификационными данными может быть осуществлен. После успешной проверки состояния система переходит к отправке запроса на осуществление платежа </p></td>
</tr>

<tr>
<td><p>7</p></td>
<td><p>Транзакция для данного пользователя запрещена по техническим причинам</p></td>
</tr>

</tbody>
</table>

##Запрос на осуществление Перевода
###Pay
Когда Плательщик успешно совершает оплату через сервис, Pericles отправляет детали проведенного платежа на URL платежного приложения, указанный в личном кабинете. Запрос отправляется методом GET и содержит следующие параметры:

<b>Параметры запроса</b>
<table border="1" cellspacing="0" cellpadding="0" > <thead>

<tr>
<td><p>Название поля</p></td>
<td><p>Тип</p></td>
<td><p>Описание</p></td>
<td><p>Обязательность</p></td>
<td><p>Пример</p></td>
</tr>
<tbody>
<tr>
<td><p>сommand</p></td>
<td><p>String</p></td>
<td><p>Признак того, что идет оповещение о Переводе</p></td>
<td><p>Да</p></td>
<td><p>pay</p></td>
</tr>

<tr>
<td><p>id</p></td>
<td><p>String</p></td>
<td><p>Уникальный id операции в системе</p></td>
<td><p>Да</p></td>
<td><p>7555545</p></td>
</tr>

<tr>
<td><p>v1</p></td>
<td><p>String(255 символов)</p></td>
<td><p>Параметр идентификации Плательщика (например, ник или e-mail)</p></td>
<td><p>Нет</p></td>
<td><p>User</p></td>
</tr>

<tr>
<td><p>v2</p></td>
<td><p>String(200 символов)</p></td>
<td><p>Дополнительный параметр идентификации Плательщика (в зависимости от настроек проекта)</p></td>
<td><p>Нет</p></td>
<td><p>0</p></td>
</tr>

<tr>
<td><p>v3</p></td>
<td><p>String(100 символов)</p></td>
<td><p>Дополнительный параметр идентификации Плательщика (в зависимости от настроек проекта)</p></td>
<td><p>Нет</p></td>
<td><p>0</p></td>
</tr>

<tr>
<td><p>sum</p></td>
<td><p>Float</p></td>
<td><p>Сумма денежных средств. Разделитель "." (2 знака после точки)</p></td>
<td><p>Да</p></td>
<td><p>100.98</p></td>
</tr>

<tr>
<td><p>test</p></td>
<td><p>Integer</p></td>
<td><p>Признак тестовой транзакции. test=1 — система проводит тестовую транзакцию. Реального платежа не было. test=0 — реальный платеж</p></td>
<td><p>Нет</p></td>
<td><p>1</p></td>
</tr>

<tr>
<td><p>md5</p></td>
<td><p>String</p></td>
<td><p>Подпись для предотвращения несанкционированного доступа</p></td>
<td><p>Да</p></td>
<td><p>9286b1ff8c52 26b666a20dd b4cc03c2b</p></td>
</tr>

</tbody>
</table>

###Правила формирования подписи md5

Подпись служит для обеспечения безопасности при проверке и проведении платежей. Формируется с помощью алгоритма хэширования md5 от строки, полученной конкатенацией параметров, представленных ниже:
<pre>md5(pay&lt;v1>&lt;id>&lt;secret_key>)</pre>
<b>secret_key</b> — секретное слово, которое было указано в личном кабинете.
###Пример

Для формирования подписи необходимы следующие параметры:

•command=pay;

•v1=User;

•id=7555545;

•secret_key=password.

Строка будет иметь следующий вид:

<pre>payUser7555545password</pre>

md5 хэш от данной строки:

<pre>d5069857a53ac7e92e570e85ce0be593</pre>

###Проверка ip-адреса и md5 подписи
Перед обработкой запроса проект должен проверить его подлинность. Для этого необходимо проверить принадлежность IP-адреса, с которого пришел запрос, указанному диапазону адресов, а также сравнить значение параметра md5 cо значением хэша md5 от строки, полученной согласно принципу, описанному выше. Если значения не совпадают, то запрос не является подлинным и должен быть проигнорирован. Проект должен уведомить систему об ошибке. Как только подлинность уведомления была проверена, ваш сервер должен продолжить обработку.

Для проверки принадлежности IP-адреса указанным подсетям можно воспользоваться библиотекой <a href="https://github.com/mattallty/Utils-IP-Tools">Utils-IP-Tools</a>.

<b>Пример проверки подлинности запроса:</b>
 
           <pre> &lt;?php
            function verifyMD5($secretKey, $command, $v1, $v2, $v3, $id, $md5) 
            {
            $md5Hash = md5($command.$v1.$id.$secretKey);
            if(strtolower($md5Hash) != strtolower($md5))
                return(false);
                return(true);
                }
                ?>
        </pre>
            
 

###Обработка запроса на осуществление Перевода
База данных проекта не должна содержать две успешно проведенные транзакции с одним и тем же id. Если система присылает запрос повторно с уже существующим в базе id, то платежное приложение проекта должно вернуть результат обработки предыдущего запроса.

<b>Пример обработки запроса pay:</b>

 
    <?php
    $v1 = $_GET['v1'];
    $sum = $_GET['sum'];
    $id = $_GET['id'];
    // проведение Перевода
    // pay –функция, которая производит осуществление Перевода на стороне проекта; функция должна вернуть id
    $paymentId = pay($v1, $sum, $id);
    // ответ
    // если Перевод успешно совершен 
    if ($paymentId)
        echo '<?xml version="1.0" encoding="windows-1251"?>
    <response>
        <id>' . $id . '</id>
        <id_shop>' . $paymentId . '</id_shop>
        <sum>' . $sum . '</sum>
        <result>0</result>
        <comment>Success</comment>
    </response>
    ';
    else
    echo '<?xml version="1.0" encoding="windows-1251"?>
    <response>
        <id>' . $id . '</id>
        <id_shop>' . $paymentId . '</id_shop>
        <sum>' . $sum . '</sum>
        <result>1</result>
        <comment>Temporarily database error</comment>
    </response>
    ';
    // функция pay
    function pay($v1, $sum, $id) {
    // ваш код
    return $paymentId; }
    ?>
    
###Ответ
После обработки запроса следует сформировать ответ в формате XML и кодировке UTF-8.

<b>Формирование ответа</b>

            <?php
            function generateResponsePay($iResult, $strDescription, $strId, $strIdShop)
            {
        $responseXML = new SimpleXMLElement('<response></response>');

            $responseXML->addChild('result', $iResult);
            $responseXML->addChild('comment', $strDescription);
            $responseXML->addChild('id', $strId);
            $responseXML->addChild('id_shop', $strIdShop);

            header('Content-Type: text/xml; charset=utf8');
            echo $responseXML->asXML();
            }
            ?>
            
 
Схема ответа
<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    &lt;xsd:simpleType name="gameCurrencyMoney">
        &lt;xsd:restriction base="xsd:decimal">
            &lt;xsd:minInclusive value="0"/>
        &lt;/xsd:restriction>
    &lt;/xsd:simpleType>
    &lt;xsd:element name="response">
        &lt;xsd:complexType>
            &lt;xsd:all>
                &lt;xsd:element name="id" type="xsd:unsignedInt"/>
                &lt;xsd:element name="id_shop" type="xsd:string"/>
                &lt;xsd:element name="result" type="xsd:unsignedInt"/>
                &lt;xsd:element name="sum" type="gameCurrencyMoney"/>
                &lt;xsd:element name="comment" type="xsd:string" minOccurs="0"/>
            &lt;/xsd:all>
        &lt;/xsd:complexType>
    &lt;/xsd:element>
&lt;/xsd:schema></pre>

###Примеры XML
если Перевод был успешно совершен:
<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;id>7555545&lt;/id>
    &lt;id_shop>1234&lt;/id_shop>
    &lt;sum>10&lt;/sum>
    &lt;result>0&lt;/result>
&lt;/response></pre>
•id_shop — id транзакции в системе проекта.

Если совершение Перевода завершилось неудачей:

<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;id>7555545&lt;/id>
    &lt;id_shop>1234&lt;/id_shop>
    &lt;sum>10&lt;/sum>
    &lt;result>1&lt;/result>
    &lt;comment>Temporary database error&lt;/comment>
&lt;/response></pre>

•id — Уникальный id операции в Pericles;

•id_shop — id транзакции в системе проекта (обязательный параметр);

•sum — сумма денежных средств;

•result — код завершения;

•comment — описание результата.

###Список кодов завершения
 
<table border="1" cellspacing="0" cellpadding="0" ><thead> 

<tr>
<td><p>Код</p></td>
<td><p>Описание</p></td>
<td><p>Фатальность</p></td>
</tr>
</thead>
<tbody>
<tr>
<td><p>0</p></td>
<td><p>ОК</p></td>
<td><p>Нет</p></td>
</tr>

<tr>
<td><p>1</p></td>
<td><p>Временная ошибка, повторить запрос позже</p></td>
<td><p>Нет</p></td>
</tr>

<tr>
<td><p>2</p></td>
<td><p>Неверный идентификатор Плательщика</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>3</p></td>
<td><p>Неверная подпись md5</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>4</p></td>
<td><p>Неверный формат запроса (неверная сумма, неполный набор параметров)</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>5</p></td>
<td><p>Другая ошибка (желательно описать ее в <i>comment</i>)</p></td>
<td><p>Да</p></td>
</tr>

<tr>
<td><p>7</p></td>
<td><p>Транзакция для данного Плательщика запрещена по техническим причинам</p></td>
<td><p>Да</p></td>
</tr>

</tbody>
</table>

##Отмена платежа
###Cancel
Платежи по некоторым способам оплаты (например, кредитные карты, PayPal) могут быть отменены Плательщиком в течение определенного времени (в зависимости от платежной системы). При получении запроса на отмену платежа, система обязана вернуть средства в платежную систему. Это позволяет недобросовестным Плательщикам возвратить потраченные средства после совершения Перевода. Для защиты проекта возможно реализовать запрос на отмену Перевода, что позволит оповещать проект о каждом запросе пользователя на возврат средств. Это даст возможность отслеживать попытки мошенничества и принимать необходимые решения - блокировать аккаунт пользователя, отменять Перевод и т д. За дополнительной информацией обратитесь к аккаунт-менеджеру проекта.

Pericles отправляет детали платежа для отмены на URL платежного приложения, указанный в личном кабинете. Запрос отправляется методом GET и содержит следующие параметры:
<table border="1" cellspacing="0" cellpadding="0" > <thead>

<tr>
<td><p>Параметр</p></td>
<td><p>Тип</p></td>
<td><p>Описание</p></td>
<td><p>Обязательность</p></td>
<td><p>Пример</p></td>
</tr>
<tbody>
<tr>
<td><p>сommand</p></td>
<td><p>String</p></td>
<td><p>Признак выполнения отмены платежа</p></td>
<td><p>Да</p></td>
<td><p>cancel</p></td>
</tr>

<tr>
<td><p>id</p></td>
<td><p>String</p></td>
<td><p>Уникальный id операции в системе</p></td>
<td><p>Да</p></td>
<td><p>7555545</p></td>
</tr>

<tr>
<td><p>md5</p></td>
<td><p>String</p></td>
<td><p>Подпись для предотвращения несанкционированного доступа</p></td>
<td><p>Да</p></td>
<td><p>e9b9777e9c0a 4595ad009eca 90ba9977</p></td>
</tr>

</tbody>
</table>

###Правила формирования подписи md5
Подпись служит для обеспечения безопасности при проверке и проведении Переводов. Формируется с помощью алгоритма хэширования md5 от строки, полученной конкатенацией параметров, представленных ниже:

<pre>md5(&lt;command&gt;&lt;id&gt;&lt;secret_key&gt;)</pre>
secret_key — секретное слово, которое было указано в личном кабинете.

###Пример
Для формирования подписи необходимы следующие параметры:

•command=cancel;
•id=7555545;
•secret_key=password;

Строка будет иметь следующий вид:

<pre>cancel7555545password</pre>

md5 хэш от данной строки:

<pre>e9b9777e9c0a4595ad009eca90ba9977</pre>

###Проверка ip-адреса и md5 подписи
Перед обработкой запроса проект должен проверить его подлинность. Для этого необходимо проверить принадлежность IP-адреса, с которого пришел запрос, указанному диапазону адресов, а также сравнить значение параметра md5 со значением хэша md5 от строки, полученной согласно принципу, описанному выше. Если значения не совпадают, то запрос не является подлинным и должен быть проигнорирован. Проект должен уведомить Pericles об ошибке. Как только подлинность уведомления была проверена, ваш сервер должен продолжить обработку.

Для проверки принадлежности IP-адреса указанным подсетям можно воспользоваться библиотекой <a href="https://github.com/mattallty/Utils-IP-Tools">Utils-IP-Tools</a>.

<b>Пример проверки подлинности запроса:</b>
 
            <?php
            function verifyMD5($secretKey, $command, $v1, $md5)
            {

        $md5Hash = md5($command.$id.$secretKey);
        if(strtolower($md5Hash) != strtolower($md5))
                return(false);

        return(true);
        }
        ?>
            
 
<b>Обработка запроса на отмену платежа</b>
 
    <?php
    $id = $_GET['id'];
    // отмена платежа
    $cancelResult = cancel($id);
    // если отмена платежа прошел успешно 
    if ($cancelResult)
        echo '<?xml version="1.0" encoding="windows-1251"?>
    <response>
        <result>0</result>
    </response>
    ';
    else
    echo '<?xml version="1.0" encoding="windows-1251"?>
    <response>
        <result>2</result>
        <comment>Payment with given ID does not exist</comment>
    </response>
    ';
    // функция cancel
    function cancel($id) {
    // ваш код
    return true; }
    ?>
    
 

###Ответ
После обработки запроса следует сформировать ответ в формате XML и кодировке UTF-8.
Формирование ответа

 
            <?php
            function generateResponseCancel($iResult, $strDescription)
            {
        $responseXML = new SimpleXMLElement('<response></response>');

            $responseXML->addChild('result', $iResult);
            $responseXML->addChild('comment', $strDescription);

            header('Content-Type: text/xml; charset=utf8');
            echo $responseXML->asXML();
            }
            ?>
            
 
###Ответ схемы
<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
&lt;xsd:element name="response">
        &lt;xsd:complexType>
            &lt;xsd:all>
                &lt;xsd:element name="result" type="xsd:unsignedInt"/>
                &lt;xsd:element name="comment" type="xsd:string" minOccurs="0"/>
            &lt;/xsd:all>
        &lt;/xsd:complexType>
    &lt;/xsd:element>
&lt;/xsd:schema></pre>

###Примеры XML
Если отмена платежа прошла успешно:
<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;result>0&lt;/result>
&lt;/response></pre>

Если процесс отмены платежа завершился неудачей:

<pre>&lt;?xml version="1.0" encoding="windows-1251"?>
&lt;response>
    &lt;result>2&lt;/result>
    &lt;comment> Payment with given ID does not exist&lt;/comment>
&lt;/response></pre>
•result — код завершения;

•comment — описание результата.


