# Домашнее задание к занятию «Организация сети»
## Подготовка к выполнению задания
Домашнее задание состоит из обязательной части, которую нужно выполнить на провайдере Yandex Cloud, и дополнительной части в AWS (выполняется по желанию).
Все домашние задания в блоке 15 связаны друг с другом и в конце представляют пример законченной инфраструктуры.
Все задания нужно выполнить с помощью Terraform. Результатом выполненного домашнего задания будет код в репозитории.
Перед началом работы настройте доступ к облачным ресурсам из Terraform, используя материалы прошлых лекций и домашнее задание по теме «Облачные провайдеры и синтаксис Terraform». Заранее выберите регион (в случае AWS) и зону.
### Задание 1. Yandex Cloud
Что нужно сделать
```
Создать пустую VPC. Выбрать зону.
Публичная подсеть.
Создать в VPC subnet с названием public, сетью 192.168.10.0/24.
Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использовать fd80mrhj8fl2oe87o4e1.
Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.
Приватная подсеть.
Создать в VPC subnet с названием private, сетью 192.168.20.0/24.
Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.
Создать в этой приватной подсети виртуалку с внутренним IP, подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.
Resource Terraform для Yandex Cloud:

VPC subnet.
Route table.
Compute Instance.
```

### Решение задания 1. Yandex Cloud

Создаем пустую VPC с именем dvl:
```
resource "yandex_vpc_network" "dvl" {
  name = var.vpc_name

variable "vpc_name" {
  type        = string
  default     = "dvl"
  description = "VPC network"
}
```
Создаю в VPC публичную подсеть с названием public, сетью 192.168.10.0/24:
```
resource "yandex_vpc_subnet" "public" {
  name           = var.public_subnet
  zone           = var.default_zone
  network_id     = yandex_vpc_network.dvl.id
  v4_cidr_blocks = var.public_cidr
}

variable "public_cidr" {
  type        = list(string)
  default     = ["192.168.10.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "public_subnet" {
  type        = string
  default     = "public"
  description = "subnet name"
}
```
Также ресурс и переменные можно посмотреть в файлах network.tf - https://github.com/slava1005/9_1/blob/main/terraform/network.tf 
и variables.tf - https://github.com/slava1005/9_1/blob/main/terraform/variables.tf

Создаю в публичной подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использую fd80mrhj8fl2oe87o4e1.
Листинг инстанса можно посмотреть в файле nat_instance.tf - https://github.com/slava1005/9_1/blob/main/terraform/nat_instance.tf

Создаю в публичной подсети виртуальную машину с публичным IP.
Листинг виртуальной машины можно посмотреть в файле public.tf - https://github.com/slava1005/9_1/blob/main/terraform/public.tf

Создаю в VPC приватную подсеть с названием private, сетью 192.168.20.0/24:
```
resource "yandex_vpc_subnet" "private" {
  name           = var.private_subnet
  zone           = var.default_zone
  network_id     = yandex_vpc_network.dvl.id
  v4_cidr_blocks = var.private_cidr
  route_table_id = yandex_vpc_route_table.private-route.id
}

variable "private_cidr" {
  type        = list(string)
  default     = ["192.168.20.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "private_subnet" {
  type        = string
  default     = "private"
  description = "subnet name"
}
```
Также ресурс и переменные можно посмотреть в файлах network.tf - https://github.com/slava1005/9_1/blob/main/terraform/network.tf
и variables.tf - https://github.com/slava1005/9_1/blob/main/terraform/variables.tf

Создаю route table. Добавляю статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс:
```
resource "yandex_vpc_route_table" "private-route" {
  name       = "private-route"
  network_id = yandex_vpc_network.dvl.id
  static_route {
    destination_prefix = "0.0.0.0/0"
    next_hop_address   = "192.168.10.254"
  }
}
```
Создаю в приватной подсети виртуальную машину с внутренним IP, внешний IP будет отсутствовать!
Листинг виртуальной машины можно посмотреть в файле private.tf - https://github.com/slava1005/9_1/blob/main/terraform/private.tf

Инициализирую проект, выполню код:

```
root@DESKTOP-QKJU13U:/home/slava/9_1/terraform# terraform init
Initializing the backend...
Initializing provider plugins...
- Finding latest version of yandex-cloud/yandex...
- Installing yandex-cloud/yandex v0.133.0...
- Installed yandex-cloud/yandex v0.133.0 (self-signed, key ID E40F590B50BB8E40)
Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
root@DESKTOP-QKJU13U:/home/slava/9_1/terraform# terraform plan
....................
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.nat will be created
  + resource "yandex_compute_instance" "nat" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hardware_generation       = (known after apply)
      + hostname                  = "nat"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMU/OIxLXN7N6G9P9jFwFHnJcjFoMHh16oY6fqZrXpg4 root@DESKTOP-QKJU13U
            EOT
        }
      + name                      = "nat"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd80mrhj8fl2oe87o4e1"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + metadata_options (known after apply)

      + network_interface {
          + index              = (known after apply)
          + ip_address         = "192.168.10.254"
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy (known after apply)

      + resources {
          + core_fraction = 5
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_compute_instance.private will be created
  + resource "yandex_compute_instance" "private" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hardware_generation       = (known after apply)
      + hostname                  = "private"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMU/OIxLXN7N6G9P9jFwFHnJcjFoMHh16oY6fqZrXpg4 root@DESKTOP-QKJU13U
            EOT
        }
      + name                      = "private"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd89aken7ea5dq223o7t"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + metadata_options (known after apply)

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = false
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy (known after apply)

      + resources {
          + core_fraction = 5
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_compute_instance.public will be created
  + resource "yandex_compute_instance" "public" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hardware_generation       = (known after apply)
      + hostname                  = "public"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMU/OIxLXN7N6G9P9jFwFHnJcjFoMHh16oY6fqZrXpg4 root@DESKTOP-QKJU13U
            EOT
        }
      + name                      = "public"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd89aken7ea5dq223o7t"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + metadata_options (known after apply)

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy (known after apply)

      + resources {
          + core_fraction = 5
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_vpc_network.dvl will be created
  + resource "yandex_vpc_network" "dvl" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "dvl"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_route_table.private-route will be created
  + resource "yandex_vpc_route_table" "private-route" {
      + created_at = (known after apply)
      + folder_id  = (known after apply)
      + id         = (known after apply)
      + labels     = (known after apply)
      + name       = "private-route"
      + network_id = (known after apply)

      + static_route {
          + destination_prefix = "0.0.0.0/0"
          + next_hop_address   = "192.168.10.254"
            # (1 unchanged attribute hidden)
        }
    }

  # yandex_vpc_subnet.private will be created
  + resource "yandex_vpc_subnet" "private" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "private"
      + network_id     = (known after apply)
      + route_table_id = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.20.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.public will be created
  + resource "yandex_vpc_subnet" "public" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "public"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 7 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + nat_instance_info = {
      + ip_external = (known after apply)
      + ip_internal = "192.168.10.254"
      + name        = "nat"
      + network     = "dvl"
      + subnet      = "public"
    }
  + private_vm_info   = {
      + ip_external = (known after apply)
      + ip_internal = (known after apply)
      + name        = "private"
      + network     = "dvl"
      + subnet      = "private"
    }
  + public_vm_info    = {
      + ip_external = (known after apply)
      + ip_internal = (known after apply)
      + name        = "public"
      + network     = "dvl"
      + subnet      = "public"
    }

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
root@DESKTOP-QKJU13U:/home/slava/9_1/terraform# terraform apply -auto-approve
..........................
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_compute_instance.nat will be created
  + resource "yandex_compute_instance" "nat" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hardware_generation       = (known after apply)
      + hostname                  = "nat"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMU/OIxLXN7N6G9P9jFwFHnJcjFoMHh16oY6fqZrXpg4 root@DESKTOP-QKJU13U
            EOT
        }
      + name                      = "nat"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd80mrhj8fl2oe87o4e1"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + metadata_options (known after apply)

      + network_interface {
          + index              = (known after apply)
          + ip_address         = "192.168.10.254"
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy (known after apply)

      + resources {
          + core_fraction = 5
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_compute_instance.private will be created
  + resource "yandex_compute_instance" "private" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hardware_generation       = (known after apply)
      + hostname                  = "private"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMU/OIxLXN7N6G9P9jFwFHnJcjFoMHh16oY6fqZrXpg4 root@DESKTOP-QKJU13U
            EOT
        }
      + name                      = "private"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd89aken7ea5dq223o7t"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + metadata_options (known after apply)

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = false
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy (known after apply)

      + resources {
          + core_fraction = 5
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_compute_instance.public will be created
  + resource "yandex_compute_instance" "public" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hardware_generation       = (known after apply)
      + hostname                  = "public"
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMU/OIxLXN7N6G9P9jFwFHnJcjFoMHh16oY6fqZrXpg4 root@DESKTOP-QKJU13U
            EOT
        }
      + name                      = "public"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd89aken7ea5dq223o7t"
              + name        = (known after apply)
              + size        = 10
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + metadata_options (known after apply)

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + placement_policy (known after apply)

      + resources {
          + core_fraction = 5
          + cores         = 2
          + memory        = 2
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_vpc_network.dvl will be created
  + resource "yandex_vpc_network" "dvl" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "dvl"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_route_table.private-route will be created
  + resource "yandex_vpc_route_table" "private-route" {
      + created_at = (known after apply)
      + folder_id  = (known after apply)
      + id         = (known after apply)
      + labels     = (known after apply)
      + name       = "private-route"
      + network_id = (known after apply)

      + static_route {
          + destination_prefix = "0.0.0.0/0"
          + next_hop_address   = "192.168.10.254"
            # (1 unchanged attribute hidden)
        }
    }

  # yandex_vpc_subnet.private will be created
  + resource "yandex_vpc_subnet" "private" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "private"
      + network_id     = (known after apply)
      + route_table_id = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.20.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.public will be created
  + resource "yandex_vpc_subnet" "public" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "public"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "192.168.10.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

Plan: 7 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + nat_instance_info = {
      + ip_external = (known after apply)
      + ip_internal = "192.168.10.254"
      + name        = "nat"
      + network     = "dvl"
      + subnet      = "public"
    }
  + private_vm_info   = {
      + ip_external = (known after apply)
      + ip_internal = (known after apply)
      + name        = "private"
      + network     = "dvl"
      + subnet      = "private"
    }
  + public_vm_info    = {
      + ip_external = (known after apply)
      + ip_internal = (known after apply)
      + name        = "public"
      + network     = "dvl"
      + subnet      = "public"
    }
yandex_vpc_network.dvl: Creating...
yandex_vpc_network.dvl: Creation complete after 3s [id=enpbk5nvbutkgdv7fpba]
yandex_vpc_route_table.private-route: Creating...
yandex_vpc_subnet.public: Creating...
yandex_vpc_subnet.public: Creation complete after 0s [id=e9bliu4j1lvfpp9c2n61]
yandex_compute_instance.public: Creating...
yandex_compute_instance.nat: Creating...
yandex_vpc_route_table.private-route: Creation complete after 1s [id=enp1oau7qv66irqp9j66]
yandex_vpc_subnet.private: Creating...
yandex_vpc_subnet.private: Creation complete after 1s [id=e9bfmb6ki53tiubmcsb3]
yandex_compute_instance.private: Creating...
yandex_compute_instance.public: Still creating... [10s elapsed]
yandex_compute_instance.nat: Still creating... [10s elapsed]
yandex_compute_instance.private: Still creating... [8s elapsed]
yandex_compute_instance.public: Still creating... [18s elapsed]
yandex_compute_instance.nat: Still creating... [18s elapsed]
yandex_compute_instance.private: Still creating... [18s elapsed]
yandex_compute_instance.public: Still creating... [28s elapsed]
yandex_compute_instance.nat: Still creating... [28s elapsed]
yandex_compute_instance.private: Still creating... [28s elapsed]
yandex_compute_instance.public: Still creating... [38s elapsed]
yandex_compute_instance.nat: Still creating... [38s elapsed]
yandex_compute_instance.private: Still creating... [38s elapsed]
yandex_compute_instance.public: Still creating... [47s elapsed]
yandex_compute_instance.nat: Still creating... [47s elapsed]
yandex_compute_instance.private: Still creating... [46s elapsed]
yandex_compute_instance.private: Creation complete after 47s [id=fhm2cnk5i6auc33ib73p]
yandex_compute_instance.public: Creation complete after 49s [id=fhm9f93hf7ev9p8b6dgl]
yandex_compute_instance.nat: Still creating... [57s elapsed]
yandex_compute_instance.nat: Still creating... [1m7s elapsed]
yandex_compute_instance.nat: Creation complete after 1m14s [id=fhmh7r9dkcd32q5naiur]

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.

Outputs:

nat_instance_info = {
  "ip_external" = "51.250.82.190"
  "ip_internal" = "192.168.10.254"
  "name" = "nat"
  "network" = "dvl"
  "subnet" = "public"
}
private_vm_info = {
  "ip_external" = ""
  "ip_internal" = "192.168.20.33"
  "name" = "private"
  "network" = "dvl"
  "subnet" = "private"
}
public_vm_info = {
  "ip_external" = "51.250.69.23"
  "ip_internal" = "192.168.10.5"
  "name" = "public"
  "network" = "dvl"
  "subnet" = "public"
}
root@DESKTOP-QKJU13U:/home/slava/9_1/terraform#
```
Инфруструктура развёрнута, скрины прилагаю:

![img1_1](https://github.com/user-attachments/assets/e18af248-c631-4329-9208-248a13445258)

![img1_2](https://github.com/user-attachments/assets/b3f319ea-4d11-4793-91d5-99ce3096e978)

![img1_3](https://github.com/user-attachments/assets/f814b4ec-23bf-4560-842c-54f10f92214d)

![img1_4](https://github.com/user-attachments/assets/bdbf3eed-737b-4f69-a064-1c7bc181abb3)

![img1_5](https://github.com/user-attachments/assets/71e6df83-a3e6-4e3b-b5f7-9e350c270427)

![img1_6](https://github.com/user-attachments/assets/f67894ce-3020-410b-9adf-364b1003c544)

![img1_7](https://github.com/user-attachments/assets/8e743bdb-4b7e-4f68-a948-5e2acb7ea6a4)


Подключусь к виртуальной машине и проверю, есть ли из неё доступ к интернету:

![img1_8](https://github.com/user-attachments/assets/e63bb729-73e2-4751-864e-fc5924a627a6)

Хост ya.ru пингуется, интернет на публичной виртуальной машине есть, сеть работает.

Для проверки доступности интернета на приватной виртуальной машине и работы NAT-инстанса скопирую свой приватный ssh ключ на публичную виртуальную машину. Далее с публичной виртуальной машины подключусь к приватной по внутреннему IP адресу:

scp /home/serg/.ssh/id_ed25519 debian@158.160.110.94:/home/debian/.ssh
id_ed25519

![img1_9](https://github.com/user-attachments/assets/1e4f19af-e90f-47b1-a77c-2fc3e68f1dc3)

Хост ya.ru пингуется, интернет на приватной виртуальной машине есть, сеть работает.

Выключу виртуальную машину с NAT-инстансом:

![img1_9a](https://github.com/user-attachments/assets/e16040a9-a5db-4338-8e12-034a0c7160da)

Проверю работу интернета на приватной виртуальной машине:

![img1_10](https://github.com/user-attachments/assets/ea2ff42d-f86d-4a7c-98a2-b1851a2db14d)

Интернет на приватной виртуальной машине перестал работать после отключения NAT-инстанса.

Включу виртуальную машину с NAT-инстансом.

![img1_11](https://github.com/user-attachments/assets/ed3f5e9b-65c6-41f2-bb05-6b5ffa72feb2)

![img1_12](https://github.com/user-attachments/assets/ae71be6c-82c1-4d04-90b7-8c4ac89279d0)

Интернет на приватной виртуальной машине снова заработал после включения NAT-инстанса, следовательно статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс был настроен корректно.

После разворачивания инфраструктуры получаем следующие виртуальные машины:

![img1_13](https://github.com/user-attachments/assets/2eb3942b-9909-441a-867f-df929f84a49d)

Output вывод Terraform выглядит следующим образом:

Outputs:
```
nat_instance_info = {
  "ip_external" = "51.250.82.190"
  "ip_internal" = "192.168.10.254"
  "name" = "nat"
  "network" = "dvl"
  "subnet" = "public"
}
private_vm_info = {
  "ip_external" = ""
  "ip_internal" = "192.168.20.33"
  "name" = "private"
  "network" = "dvl"
  "subnet" = "private"
}
public_vm_info = {
  "ip_external" = "51.250.69.23"
  "ip_internal" = "192.168.10.5"
  "name" = "public"
  "network" = "dvl"
  "subnet" = "public"
}
```
