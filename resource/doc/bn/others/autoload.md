# স্বয়ংক্রিয় লোড

## Composer-এ PSR-0 স্পেসিফিকেশন এর ফাইলগুলি তালিকাভুক্ত করার জন্য
webman স্বয়ংক্রিয়ভাবে `PSR-4` লোডিং রেগুলেশন মেনে চলে। আপনি যদি আপনার ব্যবসা এর জন্য `PSR-0` স্পেসিফিকেশন কোড লাইব্রেরি লোড করতে চান, তাহলে নিম্নলিখিত পদক্ষেপগুলি অনুসরণ করুন।

- `extend` নামক ডিরেক্টরি তৈরি করুন যেখানে `PSR-0` স্পেসিফিকেশন ভেরসন সুস্থ থাকবে
- `composer.json` ফাইল সম্পাদনা করুন, `autoload` এর নীচে নিম্নলিখিত বিষয় যুক্ত করুন।

```js
"psr-0" : {
    "": "extend/"
}
```
সমাপ্তিতে আপনার ফলাফল হোক
![](../../assets/img/psr0.png)

- `composer dumpautoload` কমান্ড চালু করুন
- `php start.php restart` কমান্ড দিয়ে webman পুনরারম্ভ করুন (দয়া করে দ্য ভায়জ প্রভাবিত হবে) 

## কিছু ফাইল লোড করার জন্য Composer বোর্ডিনত

- `composer.json` ফাইল সম্পাদনা করুন, `autoload.files` এর নিচে লোড করতে চাইবেন সেই ফাইলগুলি যুক্ত করুন
```json
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- `composer dumpautoload` কমান্ড চালু করুন
- `php start.php restart` কমান্ড দিয়ে webman পুনরারম্ভ করুন (দয়া করে দ্য ভায়জ প্রভাবিত হবে) 

> **পরামর্শ**
> composer.json ফাইলে যে ফাইলগুলি ছিল `autoload.files` সে ফাইলগুলি যখন অ্যাপ চালুহবার আগে লোড হয়। কিন্তু ফ্রেমওয়ার্ক দ্বারা `config/autoload.php` লোড করা ফাইলগুলি অ্যাপ চালুহবার পরে লোড হয়। 
> composer.json এ যুক্তিযুক্ত ফাইল পরিবর্তন পাসে 'restart' করতে হবে, রিলোড হবেনা। `config/autoload.php` দ্বারা লোড হওয়া ফাইলগুলি সাপোর্ট করবে হট লোডিং, পরিবর্তন করলে 'reload' দিয়ে পানিরূপ হয়েই যাবে।


## কিছু ফাইল লোড করার জন্য ফ্রেমওয়ার্ক বোর্ডিনত

এমন কিছু ফাইল হতে পারে যা `SPR` স্পেসিফিকেশনের মানদণ্ড অনুযায়ী নয়, সেই ফাইলগুলি লোড করার জন্য আমরা `config/autoload.php` ফাইল কনফিগার করতে পারি, উদাহরণস্বরূপ - 
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **পরামর্শ**
 > আমরা দেখতে পাচ্ছি `autoload.php` ফাইলে সেট করা আছে লোড `support/Request.php` এবং `support/Response.php` দুটি ফাইল, যেসব কারণে `vendor/workerman/webman-framework/src/support/` ডিরেক্টরিতে আছে দুটি একই নামের ফাইল, আমরা এই 'autoload.php' এর দ্বারা প্রাথমিকভাবে আমাদের প্রোজেক্টের মূল ডিরেক্টরি থেকে লোড করেছি `support/Request.php` এবং `support/Response.php`, এটা আমাদের এই দুটি ফাইলের অনুরূপতা নিয়ন্ত্রণ করার অংশ। আপনি এগুনি এগুনি সেটিংস ঘোষাপত্র করতে চাইলে আপনি এই দুটি কনফিগারেশন উপেক্ষা করতে পারেন।