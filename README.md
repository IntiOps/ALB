# ALB
Descripcion de Recursos y providers para despliegue de ALB  y modulos en terraform
Módulos en Terraform
¿Qué son los módulos?
Podemos entenderlo como un conjunto de ficheros de terraform que, al ser aplicados con ciertos argumentos, aprovisionan los recursos en la nube y generan una información de salida.
Existen ficheros con extensión .tf dentro de un directorio como un modulo
Lo que debemos tener en cuenta que existen argumentos de entradas y de salidas como son los inputs y los outputs. 



¿Cómo funcionan los módulos?

 
En lugar de crearlo todo desde un solo archivo se podrá hacer desde pequeños fragmentos. En una estructura de archivos similar a la siguiente.


Crear recursos en terraform 
 
 Primero crearemos dos archivos con la extensión .tf en un folder vacío. Uno con el nombre main.tf y otro llamado variables.tf.
Dentro el archivo main.tf se colocara lo siguiente :

provider "aws" {
 region = "us-west-2" 
} 
resource "aws_resource_group" "rg"{
	name = var.resource_group_name 
	tags = { 
		Name = "MyResourceGroup" } }




Y en el archivo variables.tf coloca lo siguiente.
variable "resource_group_name" { 
type = string description = "Desired name for your group" default = "testingGroup" 
}
variable "location" { 
	type = string 
	description = "Desired area location" 
	default = "us-west-2" 
} 

provider "aws" { region = var.location 
}
resource "aws_resource_group" "rg" {
 name = var.resource_group_name 
 tags = { 
      Name = "MyResourceGroup" 
	} 
}





Recursos para desplegar un ALB en AWS

Aws_lb: 
•	Se puede hacer que el balanceador de carga sea privado, al que solo se puede acceder dentro de VPC ,o hacerlo público para aceptar solicitudes de Internet.
•	Subnets: Define las subredes en las que se desplegará el balanceador de carga.
•	Enable_deletion_protection: Habilita la protección contra eliminación accidental para el balanceador de carga, estableciéndolo en true.
•	Access_logs: Configura los registros de acceso  y q Opcionalmente se le puede habilitar y almacenar en un depósito S3 .
resource "aws_lb" "test" {
  name               = "test-lb-tf"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb_sg.id]
  subnets            = [for subnet in aws_subnet.private : subnet.id]

  enable_deletion_protection = true

  access_logs {
    bucket  = aws_s3_bucket.lb_logs.id
    prefix  = "test-lb"
    enabled = true
  }

  tags = {
    Environment = "production"
  }
}
Aws_Lb_Listener
Estas entradas de información necesitan ser escuchadas y en este caso las escuchan por el puerto 80 en donde utiliza el protocolo http . cuando el trafico llega a este puerto se redirecciona al puerto 443 usando protocolos de https . 
resource "aws_lb_listener" "front_end" {
  load_balancer_arn = aws_lb.front_end.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }

}


Route 53 
Pera proteger la aplicación con https necesitaremos crear un dominio personalizado y obtener un certificado TLS 


data "aws_route53_zone" "selected" {
  name         = "test.com."
  private_zone = true
}

resource "aws_route53_record" "www" {
  zone_id = data.aws_route53_zone.selected.zone_id
  name    = "www.${data.aws_route53_zone.selected.name}"
  type    = "A"
  ttl     = "300"
  records = ["10.0.0.1"]
}



