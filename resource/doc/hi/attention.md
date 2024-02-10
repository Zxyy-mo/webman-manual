# प्रोग्रामिंग की जानकारी

## ऑपरेटिंग सिस्टम
webman एक ही समय में लिनक्स और विंडोज सिस्टम का समर्थन करता है। लेकिन workerman विंडोज के तहत एकाधिक प्रक्रिया सेटिंग और गार्ड प्रोसेस का समर्थन नहीं कर सकता है, इसलिए विंडोज सिस्टम का सिफारिश केवल डेवलपमेंट और डीबगिंग के लिए किया जाता है, और नामस्ते वातावरण में कृपया लिनक्स सिस्टम का उपयोग करें।

## शुरू करने का तरीका
**लिनक्स सिस्टम** को `php start.php start` (डीबग डीबग मोड) या `php start.php start -d` (गार्ड प्रोसेस मोड) से शुरू करे।
**विंडोज सिस्टम** में `windows.bat` चलाएं या `php windows.php` को शुरू करें, ctrl c दबाकर रोकें। विंडोज सिस्टम stop reload status reload connections इत्यादि कोमांड का समर्थन नहीं करता है।

## बासीत मेंटनेंस
webman एक बासीत आंतरिक स्मृति फ्रेमवर्क है, सामान्य रूप से, php फ़ाइल को आंतरिक स्मृति में लोड करने के बाद फिर से डिस्क से पढ़ने की जरूरत नहीं होती (टेम्पलेट फ़ाइल को छोड़कर)। इसलिए नामस्ते वातावरण कोड या कॉन्फ़िगरेशन परिवर्तन होने के बाद केवल `php start.php reload` को दुहराने की आवश्यकता है। अगर प्रक्रिया संबंधित सेटिंग या एक नए कॉम्पोज़र पैकेज का इंस्टॉलेशन कर रहा है तो `php start.php restart` की आवश्यकता होती है।

> विकास को आसान बनाने के लिए, वेबमैन में एक मॉनिटर स्वनिर्धारित प्रोसेस होता है जो व्यापार फ़ाइल अपडेट की निगरानी करता है, जब व्यापार फ़ाइल अपडेट होती है तो स्वचालित रूप से पुनरारंभ होता है। यह सुविधा केवल workerman डीबग मोड में (शुरू करते समय `-d` न लगाएं) होती है। विंडोज़ उपयोगकर्ताओं को इस्तेमाल करने के लिए `windows.bat` या `php windows.php` को चलाना होगा।

## आउटपुट स्टेटमेंट के बारे में
पारंपरिक php-fpm परियोजनाओं में, `echo` `var_dump` जैसे फ़ंक्शन विश्व को सीधे पृष्ठ पर दिखाई जाती है, जबकि webman में, यह आउटपुट आमतौर पर पेज पर नहीं दिखाई देता है, बल्कि वह टर्मिनल पर प्रदर्शित होता है (टेम्पलेट फ़ाइल में आउटपुट को अलग किया जाता है)।

## `exit` `die` स्टेटमेंट का न चलाना
`die` या `exit` चलाने से प्रक्रिया बंद हो जाएगी और पुनरारंभ होगी, जिससे वर्तमान अनुरोध को सही ढंग से नहीं प्रत्युत्तर दिया जा सकता है।

## `pcntl_fork` फ़ंक्शन न चलाना
`pcntl_fork` से एक प्रक्रिया बनाई जा सकती है, जो webman में अनुमति नहीं है।