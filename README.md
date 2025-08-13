# Projeto-1-Linux
Infraestrutura web local e monitoramento automatizado


# Ambiente de Virtualização Utilizado


Para este projeto, o ambiente foi configurado utilizando a seguinte estrutura:

- Virtualizador: VirtualBox 7.0.14-161095
- Sistema Operacional da Máquina Virtual: Linux Mint 22.1 Cinnamon (64 bits)

# Configuração da Rede da Máquina Virtual
A placa de rede da máquina virtual foi configurada no modo Bridge, isso permite que a máquina virtual se comunique diretamente com outros dispositivos na mesma rede


# Instalação e Configuração do Servidor Nginx
A instalação do Nginx no Linux Mint foi realizada da seguinte forma:

Passo 1: Atualizar a lista de pacotes
    
    sudo apt update

Passo 2: Instalar o Nginx

    sudo apt install nginx

Passo 3: Ativar o Nginx para iniciar automaticamente com o sistema

    sudo systemctl enable nginx

Passo 4: Testar a configuração do Nginx

    sudo nginx -t

Se a saída for "syntax is ok" e "test is successful", o servidor está pronto para uso.

Para verificar se o Nginx está ativo, utilize:

    sudo systemctl status nginx

# Criação da Página web

Após instalar e configurar o Nginx, o próximo passo foi criar uma página para exibição no servidor.

Passo 1: Acessar o diretório padrão do Nginx

    cd /var/www/html

O caminho /var/www/html é a raiz padrão utilizada pelo Nginx para servir páginas na web.
Qualquer arquivo HTML colocado aqui poderá ser acessado pelo navegador.

Passo 2: Criar um arquivo HTML

    sudo nano index.html

Passo 3: Inserir o conteúdo da página

    <!DOCTYPE html>
    <html lang="pt-BR">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Página do Chialle</title>
        <style>
            body {  
                font-family: Arial, sans-serif;
                background: linear-gradient(135deg, #4facfe, #fff);  #Estilização da página
                margin: 0;
                padding: 0;
                color: #fff;
                display: flex;
                justify-content: center;
                align-items: center;
                height: 100vh;
                text-align: center;
            }

            .container {
                background: rgba(0, 0, 0, 0.3);
                padding: 40px;
                border-radius: 15px;
                box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
                max-width: 500px;
            }

            h1 {
                margin-bottom: 15px;
                font-size: 2.5em;
            }

            p {
                font-size: 1.7em;
            }

            .container:hover {
                transform: scale(1.02);
                transition: all 0.3s ease;
            }
        </style>
    </head>
    <body>
        <div class="container">         #Conteudo a ser exibido 
            <h1>Projeto 1 - Linux</h1>
            <p>Olá Turma!
            <p>Essa é minha página HTML servida pelo Nginx.</p>
        </div>
    </body>
    </html>

Passo 4: Testar a página no navegador.
No computador host, abra um navegador.

Digite o IP da máquina virtual no campo de endereço.

A página criada deverá ser exibida.


# Criação do Webhook e Script de Monitoramento

Nesta etapa, foi implementado um sistema de monitoramento para verificar se o servidor está online e enviar alertas para o Discord caso o site fique indisponível.

Passo 1: Criar um Webhook no Discord
1. No Discord, crie um *canal de texto* para receber os alertas.
2. Vá até as configurações do canal (Editar Canal).
3. Selecione *Integrações* e *Webhooks*.
4. Clique em *Novo Webhook*.
5. Copie a URL gerada, ela será usada pelo script para enviar notificações.


Passo 2: Criar o script de monitoramento
    
    sudo nano /usr/local/bin/script.sh

Diferente de como foi feito para criar a página em html, outro método é detalhar o caminho absoluto onde quer criar e por fim o nome do arquivo.

A extensão .sh indica que o arquivo é um script escrito para o interpretador Bash.

Passo 3: Código do script

    # Configurações
    URL="http://192.168.3.51"   # Troque pelo endereço do seu site
    WEBHOOK_URL="https://discord.com/api/webhooks/XXXXXXXX/XXXXXXXX" #Troque pela URL do seu Webhook
    LOG_FILE="/var/log/monitor.log"

    # Testa se o site responde 
    STATUS=$(curl -s --max-time 5 -o /dev/null -w "%{http_code}" "$URL")

    # Data e hora
    DATA=$(date '+%Y-%m-%d %H:%M:%S')

    # Verifica se o status HTTP é 200
    if [ "$STATUS" != "200" ]; then
        MSG="Atenção!!!⚠️ ($DATA) O seu site $URL está indisponível! Código HTTP: $STATUS"

    # Envia mensagem para o Discord
        curl -H "Content-Type: application/json" \
             -X POST \
             -d "{\"content\": \"$MSG\"}" \
             "$WEBHOOK_URL"

    # Salva log
        echo "$MSG" >> "$LOG_FILE"

    # Reinicia o Nginx automaticamente
        sudo systemctl restart nginx
    else
        echo "Online!!✅ [$DATA] O seu site $URL está disponível. Código HTTP: $STATUS" >> "$LOG_FILE"
    fi


Passo 4: Dar permissão de execução ao script

    sudo chmod +x /usr/local/bin/script.sh
    
Passo 5: Configuração do sudoers

    sudo visudo

    XXXXXX=(ALL) NOPASSWD: /bin/systemctl restart nginx #Coloque seu usuário no campo "XXXXXX"

Esse comando vai permitir que o Nginx reinicie, caso esteja inativo, sem necessitar colocar a sua senha.

    
Passo 6: Configurar no Cron para rodar a cada 1 minuto.
Abra o cron:
    
    crontab -e

E adicione:

    * * * * * /usr/local/bin/script.sh


Se o site estiver fora do ar, você receberá:

-Uma notificação no Discord.

-Um registro no arquivo /var/log/monitor.log.

-O Nginx será reiniciado automaticamente para voltar ao ar.


# Funcionamento

Dessa forma o monitoramento deve estar tudo ok, para testar será necessário utilizar o comando:

    
    sudo systemctl stop nginx #Parar o servidor
 
 Seguido do:

    sudo system status nginx #Verificar se o servidor está ativo

<img width="1124" height="322" alt="status2" src="https://github.com/user-attachments/assets/936e7143-b47a-472e-ae70-a3a156720c37" />


É perceptivel que o servidor está fora do ar. Se tentarmos acessar nossa página web, não será possível.


<img width="1277" height="600" alt="pagina2" src="https://github.com/user-attachments/assets/59d86984-f98a-45f8-8f08-2e1c3adc7527" />


Automaticamente o discord recebe o alerta de que o servidor caiu e continuará recebendo a cada vez que o script rodar (1 - 1 minuto) até que seja resolvido.


<img width="720" height="404" alt="image" src="https://github.com/user-attachments/assets/e9c0f519-b337-4cb4-9855-7bb01ec38189" />


Com isso o script irá rodar novamente, fazendo com que o Nginx reinicie e volte a disponibilidade sem precisar executar nenhum movimento de forma manual.


<img width="1014" height="322" alt="status3" src="https://github.com/user-attachments/assets/54dc43c4-155d-4ef1-a0d6-d24dd8ab159a" />


É notório que o último comando antes de verificar o status novamente do servidor foi o "stop", comprovando que de forma automatizada o monitoramento foi bem sucedido, além da inclusão da reinicialização do serviço.
Por fim, a página está disponível para acesso.


<img width="1279" height="601" alt="Pagina1" src="https://github.com/user-attachments/assets/11f08aed-99f6-4d20-9f5d-511773b228a1" />
