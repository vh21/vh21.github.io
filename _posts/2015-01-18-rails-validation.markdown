---
layout: post
title: 'rails資料驗證'
date: 2015-01-18 13:31
comments: true
---

# 輸入資料的前置處理
在收到Post資料時，可透過`before_validation` callback來清理資料或設定預設值，如清除前後空白

# 驗證資料
透過validates

```ruby
validates :department, :presence => true  # 確保存在
                       :inclusion => { :in => %w(BU1 BU2) # 選項
                                       :message => "%{value} is not a valid department name"} # 錯誤訊息
```

# Reference
1. [Ruby on Rails 實戰聖經: ActiveRecord - 資料驗證及回呼](https://ihower.tw/rails4/activerecord-lifecycle.html)
2. [Stack overflow: custom error message for inclusion validation](http://stackoverflow.com/questions/6894273/custom-error-message-for-inclusion-validation)
3. [rails api: validates](http://api.rubyonrails.org/classes/ActiveModel/Validations/ClassMethods.html#method-i-validates)
