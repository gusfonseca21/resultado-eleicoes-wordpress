#vanillajs #wordpress

Os arquivos que estão na pasta wordpress/ são os códigos que efetivamente serão usados no WordPress, enquanto que os presentes em desenvolvimento-local/ são usados para o desenvolvimento pois facilitam a visualização das mudanças.

Em um arquivo HTML podemos adicionar código JavaScript e CSS, porém, ao adicionar um código HTML com script JS pelo Elementor, por exemplo, o WordPress bloqueia a execução do código JS por motivos de segurança. Para fazer a adição de código JS customizado em uma página no WordPress precisamos seguir alguns passos.

### Passo 1: criar um tema filho (_child theme_)

**OBS:** no caso da implantação em produção, não foi preciso criar um tema filho para fazer a implantação do código, já que o nosso tema não é atualizado.

Os temas (no menu de **Aparência**) são importantes pois será neles que poderemos adicionar e gerenciar código JavaScript customizado. É importante criar um tema filho pois ele herdará o tema definido pelo pai e quando o tema principal for atualizado, a nossa modificação de importação do código JS não será sobrescrito.

Na pasta `wp-content/themes/` crie uma nova pasta. O nome da pasta deve ser o nome da pasta do tema pai com "-child" no final. Por exemplo: `cef-child`.

Copie os arquivos de código dos arquivos **_style.css_** e **_functions.php_** do tema pai e cole na pasta do tema filho.

Dentro de **_style.css_** na pasta do tema filho apague todo o conteúdo e coloque o seguinte código (lembre-se que o código a seguir é somente uma referência, substitua com base no conteúdo do tema utilizado):

`/*`
`Theme Name: Inc Digital Child`
`Theme URI: https://incdigital.com.br/`
`Author: Inc Digital`
`Author URI: https://incdigital.com.br/`
`Description: Our child theme`
`Version: 2021`
`License: GNU General Public License v2 or later`
`License URI: http://www.gnu.org/licenses/gpl-2.0.html`
`Text Domain: incdigital`
`Template: cef`
`This theme, like WordPress, is licensed under the GPL.`
`Use it to make something cool, have fun, and share what you've learned with others.`
`*/`

Em **Theme Name** e **Description** coloque informações para facilitar a identificação do tema filho.

Preste a atenção nos campos **Text Domain** e **Template**, eles são a referência para o tema pai. Em **Text Domain** o valor deve ser o nome do tema instalado. Em **Template** o valor deve ser o nome da pasta do tema pai.

Agora, no arquivo **_functions.php_** do tema filho apague todo o conteúdo e adicione o seguinte código:

```php
<?php
function my_theme_enqueue_styles() {
    $parent_style = 'NOME_PASTA_TEMA_PAI-style';
    wp_enqueue_style( $parent_style, get_template_directory_uri() . '/style.css' );
    wp_enqueue_style( 'child-style',
        get_stylesheet_directory_uri() . '/style.css',
        array( $parent_style ),
        wp_get_theme()->get('Version')
    );
}
add_action( 'wp_enqueue_scripts', 'my_theme_enqueue_styles' );
```

Em **NOME_PASTA_TEMA_PAI**, seguindo nosso exemplo onde o nome da pasta do tema pai é **cef**, o valor será: `cef-style`.

Agora basta ativar o tema filho em **Aparência** -> **Temas**.

Para mais detalhes sobre como criar um tema filho, visite: https://wordpress.com/support/themes/child-themes/

### Passo 2: adicionar o código JavaScript dentro do diretório

Agora precisamos que o código JS esteja disponível dentro do nosso ambiente WordPress. O acesso ao gerenciamento de pastas e arquivos do WordPress pode variar e pode ser necessário contatar o gerenciador do WordPress para realizar esse passo.

No caminho `/public_html/wp-content/themes/NOME_DO_TEMA_FILHO/assets/js/` adicione os arquivos que serão utilizados pelo projeto.

### Passo 3: referencie o código JS em _functions.php_

Para que seja possível utilizar o código JS, o WordPress precisa que ele seja referenciado no arquivo _functions.php_.

**Faça um backup do arquivo _functions.php_ para evitar problemas**

No menu de **Aparência**, selecione o submenu **Editor de arquivos de tema**. Em seguida selecione o tema filho para editar. Você verá as pastas e arquivos que esse tema tem disponível. Note que é possível editar os arquivos de código nessa parte, se eles já estiverem sido adicionados no servidor. Selecione o arquivos _functions.php_ para editar.

Dentro de _functions.php_ vá ao final do arquivo e adicione o seguinte código:

```php

function NOME_FUNCAO() {
	if(TAG_CONDICIONAL(ID_PAGINA)) {
		wp_enqueue_script('DESCRIÇÃO_CÓDIGO', get_template_directory_uri() . '/assets/js/NOME_ARQUIVO.js', array(), "CACHE_BUSTING", false);
	}
}

add_action('wp_enqueue_scripts', 'NOME_FUNCAO');
```

- **_NOME_FUNCAO_**: Defina o nome da função em

- **_TAG_CONDICIONAL_**: Escolha a tag condicional que melhor condiz com o problema que deseja resolver. Essa tag pode servir para escolher uma página específica aonde será importado o código JS, para definir que ele será importado na página principal, entre outros. Para ver quais tags condicionais estão disponíveis, veja: https://developer.wordpress.org/themes/basics/conditional-tags/

- **_ID_PAGINA_**: _TAG_CONDICIONAL_ é um método, que pode aceitar um argumento. Se, por exemplo, selecionarmos a tag condicional **is_page()**, significa que queremos que o código JS seja importado em uma página específica. Para isso passamos como argumento o ID da página. Para saber qual o ID da página, vá no menu principal do WordPress, selecione **Páginas** e busque a a página que deseja. Ao clicar nela, você verá na URL do navegador o endereço da página com a seguinte query string `?post=ID`. Id será passado como argumento em is_page().

- **_DESCRIÇÃO_CÓDIGO_**: Faça uma descrição da funcionalidade do código.

- **_NOME_ARQUIVO_**: Nessa parte você verá `get_template_directory_uri() . '/assets/js/NOME_ARQUIVO.js'` . Esta parte serve para adicionar o caminho do arquivo a ser importado dentro do servidor. No caso específico utilizamos `get_template_directory_uri() .` para concatenar as strings do caminho do código.

- **_CACHE_BUSTING_**: Cache Butsting é uma estratégia para garantir que os usuários do site sempre tenham a versão mais atual dos arquivos, evitando que a versão do arquivo seja puxado pelo cache, em vez da versão mais atual. Para mais informações sobre essa estratégia, veja: https://www.keycdn.com/support/what-is-cache-busting. É importante saber que, apesar do artigo não recomendar o método de query string para realizar o cache busting, essa é a estratégia utilizada pelo WordPress. **TODA A VEZ QUE REALIZAR ALGUMA ALTERAÇÃO NO CÓDIGO JAVASCRIPT, É NECESSÁRIO GARANTIR QUE O CACHE BUSTING OCORRA, MODIFICANDO A STRING**.

Para mais informações sobre o método **_wp_enqueue_script_**, veja: https://developer.wordpress.org/reference/functions/wp_enqueue_script/.

### Passo 4: verifique se o código JS está corretamente importado

Acesse a sua nova página criada e acesse o código fonte da página. Procure por alguma tag como `<script type="text/javascript" src="LINK_CODIGO_JS"></script>` . Clique no link do código e verifique se o código JS está correto.

### Passo 4: adicione o código HTML à sua página

Na nova página criada, usando o Elementor ou outra ferramenta, adicione a base do seu código HTML. É importante notar que na pré-visualização da página no Elementor, o código JS não é executado.

Dentro do HTML coloque os scripts de importação de bibliotecas de JavaScript e CSS. É importante adicionar o CSS dentro do HTML para ser possível visualizar as alterações feitas no CSS em tempo real durante a edição do HTML pelo Elementor.

### Passo 5: salve as alterações e teste a página
