# Домашнее задание к занятию  «Отказоустойчивость в облаке»
# - Ушаков Игорь Юрьевич

### Задание 1
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  zone = "ru-central1-a"
}

resource "yandex_vpc_network" "net" {
  name = "my-net"
}

resource "yandex_vpc_subnet" "sub" {
  name           = "my-sub"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.net.id
  v4_cidr_blocks = ["10.10.0.0/24"]
}

resource "yandex_compute_instance" "web" {
  count = 2
  name  = "web-${count.index + 1}"

  platform_id = "standard-v3"
  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd80m2h15krph9b36mfg"
      size     = 20
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.sub.id
    nat       = true
  }

  metadata = {
    user-data = <<-EOF
      #cloud-config
      packages:
        - nginx
      runcmd:
        - systemctl start nginx
    EOF
  }
}

resource "yandex_lb_target_group" "tg" {
  name = "my-tg"

  dynamic "target" {
    for_each = yandex_compute_instance.web
    content {
      subnet_id = yandex_vpc_subnet.sub.id
      address   = target.value.network_interface[0].ip_address
    }
  }
}

resource "yandex_lb_network_load_balancer" "lb" {
  name = "my-lb"

  listener {
    name = "listener"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }

  attached_target_group {
    target_group_id = yandex_lb_target_group.tg.id

    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}

output "lb_ip" {
  value = [for listener in yandex_lb_network_load_balancer.lb.listener : [for addr in listener.external_address_spec : addr.address][0]][0]
}
....
```
![задание1](https://github.com/poproshe/netology/blob/main/img/1.png)
![задание1](https://github.com/poproshe/netology/blob/main/img/2.png)
![задание1](https://github.com/poproshe/netology/blob/main/img/3.png)
