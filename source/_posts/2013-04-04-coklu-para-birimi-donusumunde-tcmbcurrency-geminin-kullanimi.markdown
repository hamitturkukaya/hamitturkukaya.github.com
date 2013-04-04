---
layout: post
title: "Çoklu para birimi dönüşümünde TcmbCurrency geminin Kullanımı"
date: 2013-04-04 14:18
comments: true
categories: 
---
Rails'da Çoklu para birimi dönüşümü (Multi Currency) için geliştirilmiş olan [money gem](https://github.com/RubyMoney/money) ve [money-rails gem](https://github.com/RubyMoney/money-rails)'i ni kullanırken geçmişe yönelik dönüşüm yapma sıkıntısı ve oranları __Türkiye Cumhuriyeti Merkez Bankası__'ndan kur almak amacıyla [google-currency gem](https://github.com/RubyMoney/google_currency)'ini uyarlanmıştır. Bu gem ile Merkez Bankası'ndaki 20'ye yakın para birimi ile dönüşümü [Money](https://github.com/RubyMoney/money) gemi altyapısıyla kullanabilirsiniz.

Öncelikle gemfile'ımıza __money-rails__ ve __tcmb_currency__ gemlerini ekliyoruz
	gem 'money-rails'
	gem 'tcmb_currency', :git => 'git://github.com/lab2023/tcmb_currency.git'

ve ardından _bundle install_ komutunu çalıştırarak gemleri projeye dahil ediyoruz.
Gemler yüklenip, projeye dahil edildikten sonra terminalden

	$ rails g tcmb_currency:initializer
	$ rails g tcmb_currency:migration
	$ rake db:migrate

komutlarını çalıştırıp initializer dosyasını ve database tablolarını oluşturuyoruz.

Son olarak ise

	$ rake tcmb_currency:insert_from_tcmb

rake task'ını günlük olarak çalışacak bir cron job a atayarak (bu iş için [whenever gem](https://github.com/javan/whenever) kullanılabilir), günlük olarak oranların database'e eklenmesi sağlanır

Ardından tek yapılması gereken money gemi işlemleri cent,kuruş vb. bazlı yaptığı için modelinize _monetize :price_cents_ eklemek.

```ruby
class Product < ActiveRecord::Base
attr_accessible :price, :product ,:price_cents, :price_currency
monetize :price_cents
end
```

Artık view katmanında
```erb
<% @products.each do |product| %>
<tr>
<td><%= product.product %></td>
<% price =Money.new(product.price_cents,product.price_currency) %>
<td><%= humanized_money_with_symbol price %></td>
<td><%= humanized_money_with_symbol price.exchange_to(:JPY) %></td>
<td><%= humanized_money_with_symbol price.exchange_to(:EUR, "2013-03-06") %></td>
<td><%= link_to 'Show', product %></td>
<td><%= link_to 'Edit', edit_product_path(product) %></td>
<td><%= link_to 'Destroy', product, method: :delete, data: { confirm: 'Are you sure?' } %></td>
</tr>
<% end %>
```
şeklinde kullanabilirsiniz

```ruby
Money.new(1000,"USD").exchange_to(:EUR) # O güne ait oranlara göre dönüşüm yapar
Money.new(1000,"USD").exchange_to(:EUR, "2013-03-02") # Verilen tarihe ait oranlara göre dönüşüm yapar
```
