# बैंकिंग ऐप पार्ट 2 बनाएँ: एक लॉगिन और पंजीकरण फॉर्म बनाएँ

## पूर्व व्याख्यान प्रश्नोत्तरी

[पूर्व व्याख्यान प्रश्नोत्तरी](https://wonderful-flower-063e19f0f.1.azurestaticapps.net/quiz/43?loc=hi)

### परिचय

लगभग सभी आधुनिक वेब ऐप्स में, आप अपना निजी स्थान रखने के लिए एक खाता बना सकते हैं। चूंकि एक ही समय में कई उपयोगकर्ता वेब ऐप तक पहुंच सकते हैं, इसलिए आपको प्रत्येक उपयोगकर्ता के व्यक्तिगत डेटा को अलग से संग्रहीत करने के लिए एक तंत्र की आवश्यकता होती है और जानकारी प्रदर्शित करने के लिए कौन सी जानकारी का चयन करना चाहिए। हम [उपयोगकर्ता पहचान को सुरक्षित](https://en.wikipedia.org/wiki/Authentication) रूप से प्रबंधित करने के लिए को कवर नहीं करेंगे  क्योंकि यह अपने आप में एक व्यापक विषय है, लेकिन हम सुनिश्चित करेंगे कि प्रत्येक उपयोगकर्ता एक बनाने में सक्षम है (या अधिक) हमारे ऐप पर बैंक खाता।

इस भाग में हम अपने वेब ऐप में लॉगिन और पंजीकरण को जोड़ने के लिए HTML रूपों का उपयोग करेंगे। हम देखेंगे कि डेटा को सर्वर एपीआई को प्रोग्रामेटिक रूप से कैसे भेजा जाए, और अंततः उपयोगकर्ता इनपुट के लिए बुनियादी सत्यापन नियमों को कैसे परिभाषित किया जाए।

### शर्त

इस पाठ के लिए आपको वेब ऐप का [HTML टेम्प्लेट और रूटिंग](../../1-template-route/translations/README.hi.md)) पूरा करना होगा। आपको स्थानीय रूप से [Node.js](https://nodejs.org) और [सर्वर एपीआई चलाने](../../api/README.hi.md) स्थापित करने की आवश्यकता है ताकि आप खाते बनाने के लिए डेटा भेज सकें।

आप परीक्षण कर सकते हैं कि सर्वर टर्मिनल में इस कमांड को निष्पादित करके ठीक से चल रहा है:

```sh
curl http://localhost:5000/api
# -> should return "Bank API v1.0.0" as a result
```

---

## फोरम और कोन्टरोल्स

`<form>` एलेमेन्ट एक HTML दस्तावेज़ के एक भाग को एन्क्रिप्ट करता है जहां उपयोगकर्ता इनपुट कर सकता है और इंटरैक्टिव नियंत्रणों के साथ डेटा जमा कर सकता है। सभी प्रकार के उपयोगकर्ता इंटरफ़ेस (UI) नियंत्रण हैं जिनका उपयोग एक फॉर्म के भीतर किया जा सकता है, सबसे आम है `<input>` और `<button>` एलेमेन्ट ।

`<input>` के विभिन्न [प्रकार](https://developer.mozilla.org/docs/Web/HTML/Element/input) के कई उदाहरण हैं, उदाहरण के लिए एक फ़ील्ड बनाने के लिए जहां उपयोगकर्ता आप उपयोग कर सकते हैं इसके उपयोगकर्ता नाम दर्ज कर सकते हैं:

```html
<input id="username" name="username" type="text">
```

जब फॉर्म डेटा को भेज दिया जाएगा तो `name` विशेषता को संपत्ति के नाम के रूप में उपयोग किया जाएगा। `id` विशेषता का उपयोग फॉर्म नियंत्रण के साथ एक `label` को जोड़ने के लिए किया जाता है।

> 
एक विचार पाने के लिए [`<input>` प्रकार](https://developer.mozilla.org/docs/Web/HTML/Element/input) और [अन्य फोरम कोन्टरोल्स](https://developer.mozilla.org/docs/Learn/Forms/Other_form_controls) की संपूर्ण सूची पर एक नज़र डालें अपने UI का निर्माण करते समय आप उपयोग कर सकते हैं सभी देशी UI तत्व।

✅ ध्यान दें कि `<input>` एक [खाली एलेमेन्ट](https://developer.mozilla.org/docs/Glossary/Empty_element) है, जिस पर आपको एक मिलान समापन टैग नहीं जोड़ना चाहिए। हालाँकि आप स्व-समापन `<input/>` संकेतन का उपयोग कर सकते हैं, लेकिन इसकी आवश्यकता नहीं है।

फॉर्म के भीतर `<button>` एलेमेन्ट थोड़ा विशेष है। यदि आप इसकी `type` विशेषता निर्दिष्ट नहीं करते हैं, तो यह स्वचालित रूप से दबाए जाने पर सर्वर को फॉर्म डेटा प्रस्तुत करेगा। यहां संभावित `type` मान दिए गए हैं:

- `submit`: एक `form` के भीतर डिफ़ॉल्ट, बटन फार्म सबमिट एक्शन को ट्रिगर करता है।
- `reset`: बटन उनके प्रारंभिक मूल्यों पर सभी फॉर्म नियंत्रणों को रीसेट करता है।
- `button`: बटन दबाए जाने पर डिफ़ॉल्ट व्यवहार न निर्दिष्ट करें। फिर आप जावास्क्रिप्ट का उपयोग करके इसे कस्टम एक्शन दे सकते हैं।

### टास्क

आइए `login` टेम्प्लेट में एक फ़ॉर्म जोड़कर शुरू करें। हमें एक *उपयोगकर्ता नाम* फ़ील्ड और एक *लॉगिन* बटन की आवश्यकता होगी।

```html
<template id="login">
  <h1>Bank App</h1>
  <section>
    <h2>Login</h2>
    <form id="loginForm">
      <label for="username">Username</label>
      <input id="username" name="user" type="text">
      <button>Login</button>
    </form>
  </section>
</template>
```

यदि आप एक करीब से देखते हैं, तो आप देख सकते हैं कि हमने यहां एक `<label>` एलेमेन्ट भी जोड़ा है। `<label>` एलेमेन्टस का उपयोग UI नियंत्रणों में एक नाम जोड़ने के लिए किया जाता है, जैसे कि हमारा उपयोगकर्ता नाम फ़ील्ड। लेबल आपके रूपों की पठनीयता के लिए महत्वपूर्ण हैं, लेकिन अतिरिक्त लाभ के साथ भी आते हैं:

- एक लेबल को फोरम कोन्टरोल्स से जोड़कर, यह उपयोगकर्ताओं को सहायक तकनीकों (जैसे कि स्क्रीन रीडर) का उपयोग करके यह समझने में मदद करता है कि उन्हें क्या डेटा प्रदान करने की उम्मीद है।
- आप संबंधित इनपुट पर सीधे ध्यान केंद्रित करने के लिए लेबल पर क्लिक कर सकते हैं, जिससे टच-स्क्रीन आधारित उपकरणों तक पहुंचना आसान हो जाता है।

> वेब पर [एक्सेसिबिलिटी](https://developer.mozilla.org/docs/Learn/Accessibility/What_is_accessibility) एक बहुत ही महत्वपूर्ण विषय है जिसकी अक्सर अनदेखी की जाती है। [अर्थ संबंधी HTML एलेमेन्टस](https://developer.mozilla.org/docs/Learn/Accessibility/HTML) के लिए धन्यवाद यदि आप इन्हें ठीक से उपयोग करते हैं तो सुलभ सामग्री बनाना मुश्किल नहीं है। सामान्य गलतियों से बचने और एक जिम्मेदार डेवलपर बनने के लिए आप (पहुंच के बारे में और अधिक पढ़ सकते हैं](https://developer.mozilla.org/docs/Web/Accessibility)।

अब हम पंजीकरण के लिए दूसरा रूप जोड़ेंगे, पिछले एक के नीचे:

```html
<hr/>
<h2>Register</h2>
<form id="registerForm">
  <label for="user">Username</label>
  <input id="user" name="user" type="text">
  <label for="currency">Currency</label>
  <input id="currency" name="currency" type="text" value="$">
  <label for="description">Description</label>
  <input id="description" name="description" type="text">
  <label for="balance">Current balance</label>
  <input id="balance" name="balance" type="number" value="0">
  <button>Register</button>
</form>
```

`value` विशेषता का उपयोग करके हम दिए गए इनपुट के लिए एक डिफ़ॉल्ट मान को परिभाषित कर सकते हैं।
सूचना यह भी है कि `balance` के इनपुट में `number` प्रकार है। क्या यह अन्य इनपुटों की तुलना में अलग दिखता है? इसके साथ बातचीत करने का प्रयास करें।

✅ क्या आप केवल कीबोर्ड का उपयोग करके फ़ॉर्म के साथ नेविगेट और इंटरैक्ट कर सकते हैं? आप वह कैसे करेंगें?

## सर्वर पर डेटा जमा करना

अब जब हमारे पास एक कार्यात्मक UI है, तो अगला चरण हमारे सर्वर पर डेटा भेजने के लिए है। चलो हमारे वर्तमान कोड का उपयोग करके एक त्वरित परीक्षण करें: यदि आप *लॉगिन* या *रजिस्टर* बटन पर क्लिक करते हैं तो क्या होता है?

क्या आपने अपने ब्राउज़र के URL अनुभाग में परिवर्तन को देखा है?

![रजिस्टर बटन पर क्लिक करने के बाद ब्राउज़र के URL का स्क्रीनशॉट बदल जाता है](../images/click-register.png)

`<form>` के लिए डिफ़ॉल्ट क्रिया [GET मेथड](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html##ecec9.3) का उपयोग करके वर्तमान सर्वर URL को फ़ॉर्म सबमिट करना है , फॉर्म डेटा को सीधे URL में जोड़ना। इस विधि में कुछ कमियाँ हैं:

- भेजा गया डेटा आकार में बहुत सीमित है (लगभग 2000 वर्ण)
- डेटा सीधे URL में दिखाई देता है (पासवर्ड के लिए महान नहीं)
- यह फ़ाइल अपलोड के साथ काम नहीं करता है

इसीलिए आप इसे [POST विधि](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5) का उपयोग करने के लिए बदल सकते हैं, जो शरीर में सर्वर को फॉर्म डेटा भेजता है HTTP अनुरोध, पिछली सीमाओं के बिना।

> जबकि POST डेटा को भेजने के लिए सबसे आम तौर पर उपयोग की जाने वाली विधि है, [कुछ विशिष्ट परिदृश्यों में](https://www.w3.org/2001/tag/doc/whenToUseGet.html) यह GET विधि का उपयोग करने के लिए बेहतर है, जब उदाहरण के लिए खोज क्षेत्र लागू करना।

### टास्क

पंजीकरण फॉर्म में `action` और `method` गुण जोड़ें:

```html
<form id="registerForm" action="//localhost:5000/api/accounts" method="POST">
```

अब अपने नाम के साथ एक नया खाता पंजीकृत करने का प्रयास करें। * रजिस्टर * बटन पर क्लिक करने के बाद आपको कुछ इस तरह से देखना चाहिए:

![उपयोगकर्ता के डेटा के साथ एक JSON स्ट्रिंग दिखाते हुए, पता localhost:5000/api/accounts पर एक ब्राउज़र विंडो](../images/form-post.png)

यदि सब कुछ ठीक हो जाता है, तो सर्वर को आपके अनुरोध का जवाब एक [JSON](https://www.json.org/json-en.html) प्रतिक्रिया के साथ देना चाहिए जिसमें खाता डेटा बनाया गया था।

✅ एक ही नाम के साथ फिर से पंजीकरण करने का प्रयास करें। क्या होता है?

## पृष्ठ को फिर से लोड किए बिना डेटा सबमिट करना

जैसा कि आपने शायद देखा है, हमारे द्वारा अभी उपयोग किए गए दृष्टिकोण के साथ एक मामूली समस्या है: फॉर्म जमा करते समय, हम अपने ऐप से बाहर निकलते हैं और ब्राउज़र सर्वर URL पर रीडायरेक्ट करता है। हम अपने वेब ऐप के साथ सभी पेज रीलोड से बचने की कोशिश कर रहे हैं, क्योंकि हम एक [सिंगल-पेज एप्लिकेशन (SPA)](https://en.wikipedia.org/wiki/Single-page_application) हैं।

पृष्ठ पुनः लोड किए बिना सर्वर को फ़ॉर्म डेटा भेजने के लिए, हमें जावास्क्रिप्ट कोड का उपयोग करना होगा। एक `<form>` तत्व की `action` प्रॉपर्टी में एक यूआरएल डालने के बजाय, आप कस्टम क्रिया करने के लिए `javascript` स्ट्रिंग द्वारा प्रचलित किसी भी जावास्क्रिप्ट कोड का उपयोग कर सकते हैं। इसका उपयोग करने का अर्थ यह भी है कि आपको कुछ कार्यों को लागू करना होगा जो पहले ब्राउज़र द्वारा स्वचालित रूप से किए गए थे:

- फॉर्म डेटा को पुनः प्राप्त करें
- फ़ॉर्म डेटा को एक उपयुक्त प्रारूप में कनवर्ट और एन्कोड करें
- HTTP रिक्वेस्ट बनाएं और इसे सर्वर पर भेजें

### टास्क

पंजीकरण फॉर्म को `action` से बदलें:

```html
<form id="registerForm" action="javascript:register()">
```

`app.js` खोलें `register` नामक एक नया फ़ंक्शन जोड़ें:

```js
function register() {
  const registerForm = document.getElementById('registerForm');
  const formData = new FormData(registerForm);
  const data = Object.fromEntries(formData);
  const jsonData = JSON.stringify(data);
}
```

यहाँ हम `getElementById()` का उपयोग कर फॉर्म एलिमेंट को पुनः प्राप्त करते हैं और फॉर्म से मान निकालने के लिए [`FormData`](https://developer.mozilla.org/docs/Web/API/FormData/) का उपयोग करते हैं। की/वैल्यू जोड़े के एक सेट के रूप में नियंत्रण। फिर हम [`Object.fromEntries()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Object/Object/fromEntries) का उपयोग करके डेटा को एक नियमित ऑब्जेक्ट में परिवर्तित करते हैं और अंत में डेटा को [JSON](https://www.json.org/json-en.html) में क्रमबद्ध करते हैं, आमतौर पर वेब पर डेटा के आदान-प्रदान के लिए उपयोग किया जाने वाला प्रारूप।

डेटा अब सर्वर पर भेजे जाने के लिए तैयार है। `CreateAccount` नामक एक नया फ़ंक्शन बनाएँ:

```js
async function createAccount(account) {
  try {
    const response = await fetch('//localhost:5000/api/accounts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: account
    });
    return await response.json();
  } catch (error) {
    return { error: error.message || 'Unknown error' };
  }
}
```

यह क्या कार्य कर रहा है? सबसे पहले, यहां `async` कीवर्ड देखें। इसका मतलब है कि फ़ंक्शन में कोड शामिल है जो [**asynchronously**](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) को निष्पादित करेगा। जब `await` कीवर्ड का उपयोग किया जाता है, तो यह एसिंक्रोनस कोड को निष्पादित करने के लिए प्रतीक्षा करने की अनुमति देता है - जैसे सर्वर प्रतिक्रिया का इंतजार यहां जारी रखने से पहले।

यहाँ `async/await` उपयोग के बारे में एक त्वरित वीडियो है:

[![परोमीसेस के प्रबंधन के लिए Async और Await](https://img.youtube.com/vi/YwmlRkrxvkk/0.jpg)](https://youtube.com/watch?v=YwmlRkrxvkk "परोमीसेस के प्रबंधन के लिए Async और Await")

> async/await के वीडियो के लिए ऊपर दी गई छवि पर क्लिक करें।

हम JSON डेटा को सर्वर पर भेजने के लिए `fetch()` API का उपयोग करते हैं। इस विधि में 2 पैरामीटर हैं:

- सर्वर का URL, इसलिए हमने यहां `//localhost:5000/api/accounts` वापस रखा है।
- अनुरोध की सेटिंग। यही कारण है कि हम अनुरोध के लिए `POST` विधि निर्धारित करते हैं और `body` प्रदान करते हैं। जैसा कि हम JSON डेटा सर्वर पर भेज रहे हैं, हमें `Content-type` हेडर को `application/json` पर सेट करने की भी आवश्यकता है, इसलिए सर्वर को पता है कि सामग्री की व्याख्या कैसे करें।

जैसा कि सर्वर JSON के साथ अनुरोध का जवाब देगा, हम JSON सामग्री को पार्स करने के लिए `await response.json()` का उपयोग कर सकते हैं और परिणामी वस्तु वापस कर सकते हैं। ध्यान दें कि यह विधि अतुल्यकालिक है, इसलिए हम यह सुनिश्चित करने के लिए कि प्रतीक्षा के दौरान किसी भी त्रुटि को भी पकड़ा जाता है, लौटने से पहले हम यहाँ `await` कीवर्ड का उपयोग करते हैं।

अब `createAccount()` कहने के लिए `register` फ़ंक्शन में कुछ कोड जोड़ें:

```js
const result = await createAccount(jsonData);
```

चूँकि हम यहाँ `await` कीवर्ड का उपयोग करते हैं, हमें रजिस्टर फंक्शन से पहले `async` कीवर्ड जोड़ना होगा:

```js
async function register() {
```

अंत में, परिणाम को जांचने के लिए कुछ लॉग जोड़ें। अंतिम कार्य इस तरह दिखना चाहिए:

```js
async function register() {
  const registerForm = document.getElementById('registerForm');
  const formData = new FormData(registerForm);
  const jsonData = JSON.stringify(Object.fromEntries(formData));
  const result = await createAccount(jsonData);

  if (result.error) {
    return console.log('An error occured:', result.error);
  }

  console.log('Account created!', result);
}
```

वह थोड़ा लंबा था लेकिन हम वहां पहुंच गए! यदि आप अपने [ब्राउज़र डेवलपर टूल] (https://developer.mozilla.org/docs/Learn/Common_questions/What_are_browser_developer_tools) खोलते हैं, और एक नया खाता आज़माते हैं, तो आपको वेब पेज पर कोई बदलाव नहीं देखना चाहिए लेकिन एक संदेश कंसोल में दिखाई देगा जो पुष्टि करता है कि सब कुछ काम करता है।

![ब्राउज़र कंसोल में लॉग संदेश दिखाते हुए स्क्रीनशॉट](../images/browser-console.png)

✅ क्या आपको लगता है कि डेटा सर्वर पर सुरक्षित रूप से भेजा जाता है? क्या होगा यदि कोई व्यक्ति अनुरोध को बाधित करने में सक्षम था? सुरक्षित डेटा संचार के बारे में अधिक जानने के लिए आप [HTTPS](https://en.wikipedia.org/wiki/HTTPS) के बारे में पढ़ सकते हैं।

## डेटा मान्य

यदि आप पहले उपयोगकर्ता नाम सेट किए बिना एक नया खाता पंजीकृत करने का प्रयास करते हैं, तो आप देख सकते हैं कि सर्वर स्थिति कोड [400 (खराब अनुरोध)](https://developer.mozilla.org/docs/Web/HTTP/Status/400#:~:text=The%20HyperText%20Transfer%20Protocol%20(HTTP,%2C%20or%20deceptive%20request%20routing).) के साथ एक त्रुटि देता है

किसी सर्वर पर डेटा भेजने से पहले, जब संभव हो, एक वैध अनुरोध भेजना सुनिश्चित करने के लिए पहले से [फॉर्म डेटा को मान्य करें](https://developer.mozilla.org/docs/Learn/Forms/Form_validation) यह एक अच्छा अभ्यास है। HTML5 फॉर्म नियंत्रण विभिन्न विशेषताओं का उपयोग करके अंतर्निहित मान्यता प्रदान करता है:

- `required`: फ़ील्ड को भरने की आवश्यकता है अन्यथा फॉर्म जमा नहीं किया जा सकता है।
- `minlength` और `maxlength`: टेक्स्ट क्षेत्रों में न्यूनतम और अधिकतम वर्णों को परिभाषित करता है।
- `min` और `max`:एक संख्यात्मक क्षेत्र के न्यूनतम और अधिकतम मूल्य को परिभाषित करता है।
- `type`: अपेक्षित डेटा के प्रकार को परिभाषित करता है, जैसे `number`, `email`, `file` या [अन्य निर्मित प्रकार](https://developer.mozilla.org/docs/Web/HTML/Element/input). यह विशेषता फॉर्म नियंत्रण के दृश्य रेंडरिंग को भी बदल सकती है.
- `pattern`: एक [रेगुलर इक्स्प्रेशन](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Regular_Expressions) पैटर्न को निर्धारित करने के लिए परीक्षण करने की अनुमति देता है कि दर्ज किया गया डेटा वैध है या नहीं।

> युक्ति: यदि आप मान्य हैं और `:valid` और `:invalid` CSS छद्म-क्लेसे का उपयोग नहीं कर रहे हैं, तो आप अपने फ़ॉर्म नियंत्रणों को अनुकूलित कर सकते हैं।s.

### टास्क

वैध नया खाता बनाने के लिए 2 आवश्यक फ़ील्ड हैं, उपयोगकर्ता नाम और मुद्रा, अन्य फ़ील्ड वैकल्पिक हैं। फॉर्म के HTML को अपडेट करें, फ़ील्ड के लेबल में `required` विशेषता और टेक्स्ट दोनों का उपयोग करके:

```html
<label for="user">Username (required)</label>
<input id="user" name="user" type="text" required>
...
<label for="currency">Currency (required)</label>
<input id="currency" name="currency" type="text" value="$" required>
```

हालांकि यह विशेष सर्वर कार्यान्वयन अधिकतम लंबाई वाले क्षेत्रों पर विशिष्ट सीमाएँ लागू नहीं करता है, यह किसी भी उपयोगकर्ता पाठ प्रविष्टि के लिए उचित सीमा को परिभाषित करने के लिए हमेशा एक अच्छा अभ्यास है।

टेक्स्ट फ़ील्ड में एक `maxlength` विशेषता जोड़ें:

```html
<input id="user" name="user" type="text" maxlength="20" required>
...
<input id="currency" name="currency" type="text" value="$" maxlength="5" required>
...
<input id="description" name="description" type="text" maxlength="100">
```

अब यदि आप *रजिस्टर* बटन दबाते हैं और एक फ़ील्ड हमारे द्वारा परिभाषित सत्यापन नियम का सम्मान नहीं करता है, तो आपको कुछ इस तरह से देखना चाहिए:

![फॉर्म जमा करने का प्रयास करते समय सत्यापन त्रुटि दिखाते हुए स्क्रीनशॉट](../images/validation-error.png)

इस तरह के सत्यापन *से पहले* किसी भी डेटा को सर्वर पर भेजने के लिए **क्लाइंट-साइड** सत्यापन कहा जाता है। लेकिन ध्यान दें कि डेटा भेजे बिना हमेशा सभी जांचों को बेहतर बनाना संभव नहीं है। उदाहरण के लिए, यदि सर्वर पर रिक्वेस्ट भेजे बिना एक ही यूज़रनेम के साथ कोई खाता पहले से मौजूद है तो हम यहाँ जाँच नहीं कर सकते। सर्वर पर निष्पादित अतिरिक्त सत्यापन को **सर्वर-साइड** सत्यापन कहा जाता है।

आमतौर पर दोनों को लागू करने की आवश्यकता होती है, और क्लाइंट-साइड सत्यापन का उपयोग करते समय उपयोगकर्ता को त्वरित प्रतिक्रिया प्रदान करके उपयोगकर्ता के अनुभव को बेहतर बनाता है, यह सुनिश्चित करने के लिए सर्वर-साइड सत्यापन महत्वपूर्ण है कि जिस उपयोगकर्ता डेटा में आप हेरफेर करते हैं वह ध्वनि और सुरक्षित है।

---

## 🚀 चुनौती

यदि उपयोगकर्ता पहले से मौजूद है, तो HTML में एक त्रुटि संदेश दिखाएं।

यहाँ एक उदाहरण दिया गया है कि अंतिम लॉगिन पृष्ठ स्टाइल के थोड़े समय बाद कैसा दिख सकता है:

![CSS स्टाइल जोड़ने के बाद लॉगिन पेज का स्क्रीनशॉट](../images/result.png)

## व्याख्यान उपरांत प्रश्नोत्तरी

[व्याख्यान उपरांत प्रश्नोत्तरी](https://wonderful-flower-063e19f0f.1.azurestaticapps.net/quiz/44?loc=hi)

## समीक्षा और स्व अध्ययन

डेवलपर्स ने अपने फॉर्म निर्माण प्रयासों के बारे में, विशेष रूप से सत्यापन रणनीतियों के बारे में बहुत रचनात्मक जानकारी प्राप्त की है। [CodePen](https://codepen.com) के माध्यम से देख कर विभिन्न प्रकार के प्रवाह के बारे में जानें; क्या आप कुछ दिलचस्प और प्रेरक रूप पा सकते हैं?

## असाइनमेंट

[अपने बैंक ऐप को स्टाइल करें](assignment.hi.md)