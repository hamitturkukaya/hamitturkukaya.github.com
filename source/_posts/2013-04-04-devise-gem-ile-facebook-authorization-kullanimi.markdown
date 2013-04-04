---
layout: post
title: "Devise Gem ile Facebook Authorization Kullanımı"
date: 2013-04-04 14:12
comments: true
author: Hamit Türkü Kaya
categories: omniauth
---
Bunun için öncelikle ['omniauth-facebook'](https://github.com/mkdynamic/omniauth-facebook) gemine ihtiyacımız var;
	gem 'omniauth-facebook'

gem file'ımıza gemimizi ekledikten sonra __'bundle-install'__ komutunu veriyoruz ve gem projeye dahil ettikten sonra ayarlarımızı yapmaya hazırız.

Öncelikle [Facebook Developers](https://developers.facebook.com/apps) üzerinden bir api-key alıyoruz.
ve devise initilializerina bu api keyi giriyoruz


#####config/initializers/devise.rb
```ruby
require "omniauth-facebook"
config.omniauth :facebook, 'XXXXXX', 'YYYYYYY'
```

Bu işlemden sonra istek başarılı olursa User tablomuzda tutlacak Facebook provider id için bir migration oluştruyoruz.

#####db/migrate/xxxx_add_provider_id_to_users.rb
```ruby
class AddProviderIdToUsers < ActiveRecord::Migration
def change
add_column :users, :provider_id, :string
add_index :users, :provider_id
end
end
```

migration'ımızı oluşturduktan sonra 'rake db:migrate' koutuyla değişiklikleri uyguluyoruz.

Sıradaki işlem de facebook'tan gelen callbackleri kontrl edecek controllerı yazmak bu yüzden auth_callbacks_controller.rb dosyamızı oluşturup içine kullanıcı kayıtlıysa session açan, değilse üye yapan metodumuzu yazıyoruz

#####app/controllers/auth_callbacks_controller.rb
```ruby
class AuthCallbacksController < Devise::OmniauthCallbacksController
def facebook
@user = User.find_for_facebook_oauth(request.env["omniauth.auth"], current_user)
if @user.persisted?
flash[:success] = I18n.t "devise.omniauth_callbacks.success", :kind => "Facebook"
sign_in_and_redirect @user, :event => :authentication
else
session["devise.facebook_data"] = request.env["omniauth.auth"]
redirect_to new_user_registration_url
end
end
end
```
Son işlemimiz de User modelimizi omniauthable olarak belirtmek ve tablomuza kaydedecek methodu yazmak ve route işlemlerini yapmak olacak
bunun için önce model dosyamıza

#####app/models/user.rb
```ruby
devise :database_authenticatable, :registerable,
:recoverable, :rememberable, :trackable, :validatable, :omniauthable, :omniauth_providers => [:facebook]

def self.find_for_facebook_oauth(access_token, signed_in_resource=nil)
data = access_token.extra.raw_info
user = find_by_provider_id(data.id)
if user
user
else 
create!(email: data.email, name: data.name, password: Devise.friendly_token[0, 8], provider_id: data.id)
end
end
```
kodlarını ekliyoruz ve routes.rb dosyasındaki 'devise_for :users' satırını

```ruby
devise_for :users, :controllers => {
:omniauth_callbacks => "auth_callbacks"
}
```

şeklinde değiştiriyoruz.

Artık tek yapmamız gereken istediğimiz yerden

```ruby
= link_to "Facebook Login",user_omniauth_authorize_path(provider: 'facebook') , class: 'auth_provider'
```
linkini view katmanına eklemek. 
