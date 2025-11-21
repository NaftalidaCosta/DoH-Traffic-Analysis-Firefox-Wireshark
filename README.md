# DoH-Traffic-Analysis-Firefox-Wireshark
Análise prática comparando DNS tradicional (UDP/53) e DNS over HTTPS (DoH) no Firefox. Capturas realizadas com tcpdump e analisadas no Wireshark para visualizar diferenças de tráfego, privacidade e comportamento do TLS. Inclui consultas com nslookup, análise de SNI e documentação visual com 16 imagens.

Este projeto tem como objetivo analisar de forma prática e detalhada como o tráfego DNS se comporta **antes** e **depois** da habilitação do **DNS over HTTPS (DoH)** no navegador Firefox. A investigação foi conduzida no Kali Linux utilizando ferramentas como **tcpdump**, **Wireshark** e **nslookup**, acompanhadas de documentação visual composta por 16 imagens capturadas durante o processo.

---

## 🎯 Objetivo do Estudo

O objetivo principal foi compreender:

1. Como o tráfego DNS é transmitido sem DoH**, de forma tradicional, via UDP/53.
2. Como o tráfego DNS se altera com DoH ativado**, sendo encapsulado em conexões HTTPS (porta TCP 443).
3. Como identificar e analisar essas diferenças usando ferramentas de captura de pacotes**.
4. Como o navegador lida com consultas DNS e com o uso de resolvers HTTPS (DoH)**.
5. Evidências práticas** obtidas por meio das capturas e filtros aplicados.

---

## 🛠️ Ambiente utilizado

- Sistema operacional:** Kali Linux  
- Ferramentas de rede:** tcpdump, Wireshark, nslookup  
- Navegador: Mozilla Firefox (DNS over HTTPS habilitado no modo *Max Protection*)  
- Interface de captura: `eth1`  

---

## 🔍 Metodologia

### 1. Captura do tráfego **DNS tradicional** (sem DoH)
Antes de realizar qualquer alteração no navegador, capturei o tráfego DNS padrão:

```
$ tcpdump -i eth1 udp port 53 -w DNStrafficnoTLS.pcapng
````

Em seguida, abri a captura no Wireshark e apliquei o filtro:

<img width="1218" height="653" alt="7" src="https://github.com/user-attachments/assets/4c72ccc7-1a74-428c-baea-7d8d6f6fb654" />


```
dns.qry.name == "gist.github.com" and dns.qry.type == "A"
```

### ✔ Resultados esperados do DNS tradicional

O filtro retornou **quatro pacotes**:

* 2 pacotes de **query**
* 2 pacotes de **response**
* Todos utilizando o **registro DNS tipo A**
* Comunicação visível, em texto claro, via **UDP/53**

Essa é exatamente a característica do DNS tradicional: *não criptografado* e facilmente analisável em ferramentas como Wireshark.

---

### 2. Ativação do **DNS over HTTPS (DoH)** no Firefox

No Firefox, em:

**Settings → Privacy & Security → Enable DNS over HTTPS → Max Protection**

Esse modo força o Firefox a ignorar o DNS do sistema e usar exclusivamente resolvers HTTPS.

---

### 3. Verificação dos IPs associados aos domínios


<img width="1356" height="761" alt="12" src="https://github.com/user-attachments/assets/26a3bbe2-33c6-49cc-8dbb-a805e50fbc42" />


Usei o `nslookup` para consultar:

* O domínio **portswigger.net**
* O **CNAME github.com**

Observei:

* **1 IP** associado ao CNAME do GitHub
* **4 IPs** associados ao domínio *portswigger.net*

Essa distribuição pode indicar o uso de **load balancing** no lado da PortSwigger, uma prática comum para distribuir carga e melhorar disponibilidade.

---

### 4. Captura do tráfego após habilitar o DoH

Antes de acessar o site, iniciei uma nova captura:

```bash
$ tcpdump tcp port 443 -w DNStrafficwithTLS.pcapng
```

Em seguida, no Wireshark, filtrei:

<img width="1353" height="757" alt="16" src="https://github.com/user-attachments/assets/fd7ccd91-355f-43cb-bd2d-114f397384b1" />

O pacote com o SNI não originou-se do IP 3.166.160.90, isso indica que cada servidor tem uma responsabilidade delegada ---> Infraestrutura Loading Balancing (ILD) 
Tentei outro filtro para ser mais direto.

<img width="1354" height="761" alt="15" src="https://github.com/user-attachments/assets/e71458d4-1c42-4dd0-96d6-b88091725f4a" />


```
frame contains "portswigger.net"
```

### ✔ Resultados esperados com DoH habilitado

<img width="1343" height="758" alt="14" src="https://github.com/user-attachments/assets/e80bd365-d6b2-4986-a24a-7f23cb95a537" />


Foi possível observar:

* Um pacote contendo **SNI (Server Name Indication)** dentro do handshake TLS.
* O tráfego DNS agora não aparece mais como pacotes DNS → ele está encapsulado dentro da conexão HTTPS.
* A consulta DNS deixa de ser visível em texto claro.
* O tráfego migra de **UDP/53** para **TCP/443**.

Em outras palavras:
**o DNS passa a ser criptografado e protegido dentro do túnel TLS.**

---

## 📘 Conclusão

Este estudo demonstra de forma prática e comprovada:

* A diferença clara entre o DNS tradicional e o DNS over HTTPS.
* Como as ferramentas de análise revelam (ou deixam de revelar) detalhes das consultas DNS.
* Como o DoH melhora a confidencialidade ao eliminar a exposição do tráfego DNS no nível de rede.
* Como identificar SNI e conexões TLS relacionadas ao DoH.

Com as capturas, filtros e comparações documentadas, o projeto oferece uma base sólida para entender como o DoH opera em cenários reais.

---

## 🖼️ Documentação Visual

Incluí 16 imagens demonstrando:

* Capturas no tcpdump
* Pacotes DNS analisados
* Configuração do Firefox
* Tráfego TLS e SNI
* Comparativos entre DNS tradicional e DoH

As imagens reforçam e comprovam cada etapa descrita acima.

---

## 📁 Arquivos incluídos no repositório

* `README.md` – Este documento
* `/images` – Pasta contendo as 16 capturas de tela (não incluídas aqui)
* Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          0AC0FB1094F717F9455BB6C5775EF7261A759CD918C3CB81ED1EF5D7CAD4B693       P:\Users\*****\Pictures\images.zip


Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
MD5             DF997E7547B4B1B7FEFA524E1F87FB2F                                       P:\Users\*****\Pictures\images.zip



---

## ✍️ Autor

Projeto realizado por mim como parte de estudos avançados de análise de tráfego, protocolos criptografados e técnicas de inspeção de redes no Kali Linux.


