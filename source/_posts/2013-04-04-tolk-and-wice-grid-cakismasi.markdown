---
layout: post
title: "Tolk &amp; Wice_grid Çakışması Çözümü"
date: 2013-04-04 14:41
comments: true
author: Hamit Türkü Kaya
categories: [tolk, wice_grid, kaminari, will_paginate]
---
I18n dil dosyalarını bir web arayüzünden değiştirebilme imkanı sağlayan [tolk](https://github.com/tolk/tolk) gemini, sisteminizde [wice_grid](https://github.com/leikind/wice_grid) gemi ile birlikte kullanmaya çalıştığımızda 'undefined method' hatası alıyoruz. bunun nedeni tolk geminin kendi sayfalarını listelemek için [will_paginate](https://github.com/mislav/will_paginate), wice_grid'in ise
[kaminari](https://github.com/amatsuda/kaminari) kullanması. 
<!-- more -->
2 gemin methodları çakıştığı için sistemimiz çöküyordu. Projenin bu aşamasından sonra wice_grid'den vazgeçilemeyeceği için methodlara alias atamak ta işe yaramayınca tolk geminin güncel sürümünü kaminari'ye uyumlu hale getirelim dedik. Kaminari ile uyumlu güncel tolk gemine [buradan](https://github.com/hamitturkukaya/tolk) ulaşabilirsiniz.

Kullanım için öncelikle gemfile'ımıza

	gem 'tolk', git: 'git@github.com:hamitturkukaya/tolk.git'

gemimizi ekleyip, bundle install'la kurup projeye dahil ediyoruz.
ardından gemimizin gerekli dosyaları ve migrationları oluşturabilmesi için

	$ rake tolk:setup
	$ rake db:migrate

komutlarını çalıştırıyoruz. Bu işlemden sonra
	
	$ rake tolk:setup

komutuyla birincil dil dosyasındaki verileri tarayıp tolk geminin kullandığı tablolara kaydediyoruz. Eğer diğer dil dosyalarınızda da verileriniz varsa	

	$ rake tolk:import

komutuyla onları da içeri aktarabilirsiniz.

Artık geriye kalan tek iş `/tolk` adresine girip dil dosyalarımızı düzenledikten sonra `Apply changes` diyerek `.yml` dosyalarımızı güncellemek.

