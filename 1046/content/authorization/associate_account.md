<!---<a name="bk_CreateAzureSubscription"> </a>-->

# Associar sua conta do Office 365 ao AD do Azure para criar e gerenciar aplicativos

Para autenticar seus aplicativos, você precisa registrá-los no AD do Microsoft Azure (Active Directory do Microsoft Azure). É lá que as informações da conta de usuário do Office 365 e do aplicativo estão armazenadas. Para gerenciar o AD do Azure no Portal de Gerenciamento do Azure, você precisará de uma assinatura do Microsoft Azure. O uso do portal de gerenciamento no Microsoft Azure permite que você gerencie usuários, funções e aplicativos. 

Este artigo orienta você a associar sua conta do Office 365 ao AD do Azure para criar e gerenciar aplicativos.


## Pré-requisitos

**Conta do Office 365 para empresas**

Se você não tiver uma conta do Office 365 para empresas, poderá:

- Inscreva-se em um dos [planos do Office 365 para empresas](http://products.office.com/en-us/business/compare-office-365-for-business-plans) listados acima ou
- [Participar do Programa de Desenvolvedores do Office 365 e obter uma assinatura gratuita 1 ano do Office 365](https://aka.ms/devprogramsignup).

**Assinatura do Microsoft Azure** 

- Se você tiver uma assinatura existente do Microsoft Azure, poderá associar sua assinatura do Office 365 para empresas a ela. 

- Caso contrário, você precisará criar uma nova assinatura do Azure e associá-la à sua conta do Office 365 para registrar e gerenciar aplicativos.


<!---<a name="bk_AssociateExistingAzureSubscription"> </a>-->

## Para associar uma assinatura existente do Azure à sua conta do Office 365


1. Faça logon no [portal de Gerenciamento do Microsoft Azure](https://manage.windowsazure.com) com suas credenciais do Azure existentes (por exemplo, sua ID da Microsoft, como user@live.com).
        
2. Selecione o nó **Active Directory**, selecione a guia **Diretório** e, na parte inferior da tela, selecione **Novo**. 
     
4. No menu **Novo**, selecione **Active Directory**  >  **Diretório**  >  **Criar Personalizado**.
    
5. Em **Adicionar diretório**, na caixa suspensa **Diretório**, selecione **Usar diretório existente**. Marque **Estou pronto para sair** e escolha a marca de seleção no canto inferior direito. 
    
    Isso leva você de volta ao Portal de Gerenciamento do Azure.
        
3. Faça logon com suas informações de conta do Office 365. 
    
    Você será solicitado a responder se deseja usar seu diretório com o Azure. 
    
    **Importante** Para associar sua conta do Office 365 ao AD do Azure, você precisará de uma conta do Office 365 para empresas com privilégios de administrador global. 
    
        
4. Selecione **continuar** e **Sair agora**.
        
5. Feche o navegador e reabra o [portal](https://manage.windowsazure.com). Caso contrário, você receberá um erro de acesso negado.
    
        
6. Faça logon novamente com suas credenciais do Azure existentes (por exemplo, sua ID da Microsoft, como user@live.com). Navegue até o nó **Active Directory** e, em **Diretório**, você deverá ver agora sua conta do Office 365 listada.
    

<!--<a name="bk_AssociateNewAzureSubscription"> </a>-->

## Para criar uma nova assinatura do Azure e associá-la com sua conta do Office 365


1. Faça logon no Office 365. Na **Página inicial**, selecione o ícone **Administrador** para abrir o Centro de administração do Office 365.
2. Na página do menu no lado esquerdo, role para baixo até **Administrador** e selecione **Azure AD**.

    **Importante** Para abrir o Centro de administração do Office 365 e acessar o AD do Azure, você precisará de uma conta do Office 365 para empresas com privilégios de administrador global. 
    
3. Crie uma nova assinatura.
        
    Se você estiver usando uma versão de avaliação do Office 365, verá uma mensagem informando que o AD do Azure está limitado a clientes com serviços pagos. Você ainda pode criar uma assinatura de avaliação do Azure de 30 dias sem nenhum custo, mas precisará executar algumas etapas adicionais:
    
    1. Selecione seu país ou região e clique em **Assinatura do Azure**.
    2. Insira suas informações pessoais. Para fins de verificação, insira um número de telefone no qual você possa ser encontrado e especifique se deseja receber uma mensagem de texto ou uma chamada.
    3. Depois que você receber seu código de verificação, digite-o e clique em **Verificar código**.
    4. Insira as informações de pagamento, verifique o contrato e selecione **Inscrever-se**.
        
        Seu cartão de crédito não será cobrado.
        
        Não feche ou atualize o navegador enquanto a assinatura do Azure está sendo criada.
            
4. Depois que sua assinatura do Azure for criada, selecione **Portal**.
        
5. O Tour do Azure é exibido. Você pode exibi-lo ou escolher **X** para fechá-lo.
        
    Agora, você deve ver todos os itens na sua assinatura do Azure. Ele lista um diretório com o nome do seu locatário do Office 365.
    
