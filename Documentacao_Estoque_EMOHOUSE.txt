Aqui está o Dossiê Técnico e Funcional Definitivo do projeto Estoque EMOHOUSE.

Guarde este documento com extremo cuidado (salve como Dossie_EMOHOUSE.txt ou .md). Quando você for continuar o desenvolvimento com qualquer outra IA, você deve colar este texto no início da conversa e dizer: "Leia atentamente esta documentação de arquitetura e regras de negócio do meu aplicativo antes de propor qualquer alteração no código."

Isso garantirá que a IA compreenda o ecossistema complexo que criamos e não quebre nada que já está funcionando perfeitamente.

───

📖 DOSSIÊ TÉCNICO E ARQUITETURAL: ESTOQUE EMOHOUSE
Versão: 2.0 (Estável - Off-Cloud / JSON Based)
Tipo de Aplicação: WebApp SPA (Single Page Application) em Arquivo Único.
Objetivo: Gerenciamento doméstico avançado de inventário, auditoria de consumo e automação de lista de compras.

───

1. REGRAS INEGOCIÁVEIS DO PROJETO (Diretrizes Estritas para a IA)
Qualquer IA que for manipular este código DEVE obedecer às seguintes regras sob pena de quebrar a aplicação:

1. Single-File Architecture: O sistema inteiro (HTML, CSS customizado e JavaScript) DEVE obrigatoriamente existir em um único arquivo index.html. Não é permitido modularizar em múltiplos arquivos.
2. Zero Backend (Serverless): O sistema não possui e não deve possuir backend (Node, PHP, Python) ou banco de dados em nuvem (Firebase, SQL). Tudo roda no lado do cliente (Client-side).
3. Framework CSS: O layout utiliza Tailwind CSS via CDN. Nenhuma outra biblioteca de estilos (Bootstrap, Materialize) é permitida.
4. Manipulação de Modais (Crucial): A exibição de modais (telas sobrepostas) é controlada pela troca das classes Tailwind hidden para flex. NUNCA utilize style.display = 'block' ou style.display = 'none', pois isso destrói o alinhamento centralizado do flexbox do Tailwind. As animações de entrada usam a classe customizada .animate-fade.
5. Integrações Externas Omitidas: A integração com a API do GitHub foi REMOVIDA E BANIDA do projeto. O método de sincronização atual é inteiramente baseado na exportação/importação manual de arquivos JSON.

───

2. GERENCIAMENTO DE ESTADO E BANCO DE DADOS (appData)
O coração da aplicação é a variável global appData. Ela é persistida no localStorage sob a chave inv_data e é a exata estrutura exportada no arquivo JSON de backup.

Schema do Objeto appData:
• items (Array de Objetos): Onde ficam os produtos. 
◦ Atributos:id, name, qty (aceita string, ex: "?", "X" ou números "5.5"), alertRed (limiar numérico crítico), alertOrange (limiar de atenção), unit (unidade de medida), cat (categoria), subcat, tags (array de strings), e campos extras opcionais: expiry, establishment, price.
• categories (Array de Objetos): Formato { name: "Nome", subcats: ["Sub1", "Sub2"] }.
• tags (Array de Strings): Filtros dinâmicos criados pelo usuário.
• compendium (Array de Strings): Dicionário que salva todos os nomes já digitados para sugerir no autocomplete (tag <datalist>).
• logs (Array de Objetos): Histórico (Auditoria). Grava data, usuário, ação e detalhes da alteração comparando o "antes" e "depois" das edições. Limitado a 100 registros.
• settings (Objeto): Guarda as preferências da interface. 
◦ Controle de UI:showExpiry, showEstablishment, showPrice, enableQuickFilters, enableCatFilters.
◦ Memória de Tabela:colOrder salva a ordem exata em que o usuário organizou as colunas via Drag & Drop.
• lastUpdated (String ISO): Carimbo de tempo exato do último salvamento.

───

3. FUNCIONALIDADES CORE E COMO ESTÃO IMPLEMENTADAS

A. Pseudo-Login e Captura de Usuário
• Lógica: A tela #loginScreen usa um <form> nativo com onsubmit="event.preventDefault(); login(); return false;". Isso garante que apertar "Enter" em computadores ou no teclado virtual de smartphones acione a entrada perfeitamente sem recarregar a página (bug comum do HTML5).
• O nome é salvo em localStorage.getItem("inv_user") e anexado a cada novo log.

B. Modo Somente Visualização (Read-Only)
• Acionado pelo botão de "Olho" no header.
• Oculta todas as opções de criação, exclusão, área administrativa e remove a coluna "Ações" da tabela adicionando a classe hidden em elementos com a classe .admin-only. Ideal para não correr risco de esbarrar e deletar dados.

C. Cores de Status e Alertas de Quantidade Inteligente (Semáforo)
• Se qty for um texto puro, vazio ou interrogação (não convertível para Float), a cor da célula fica cinza (status neutro).
• Se convertível para Float, a função getItemStatus() compara com os alertas definidos individualmente dentro de cada item (alertRed e alertOrange).
• 🔴 Crítico: Abaixo do limiar vermelho.
• 🟠 Atenção: Abaixo do limiar laranja.
• 🟢 Normal: Acima dos limiares.

D. Tabela Interativa (Drag & Drop)
• Utiliza a API nativa de Drag & Drop do HTML5 (draggable="true", ondragstart, ondrop).
• Ao soltar uma coluna, o array appData.settings.colOrder é reordenado e a tabela é re-renderizada do zero. A ordem fica salva no banco de dados.

E. Sistema de Filtros (Pílulas)
• Divididos em duas instâncias isoladas: Main UI (Tabela) e Shopping List (Lista de Compras).
• Botões permitem filtrar múltiplas Categorias, Subcategorias, Tags e até Status de Cor (Vermelho, Laranja, Verde) cumulativamente.
• A barra de filtro pode ser ocultada nas Configurações.

F. Inteligência de Edição em Cascata (Lápis de Edição)
• Funcionalidade Crítica: Nos modais de gerenciar Categoria, Subcategoria, Dicionário e Tags, há um botão de Lápis. Ao renomear (ex: "Açougue" para "Carnes"), um script varre todo o appData.items e atualiza a propriedade de cada produto correspondente, garantindo que nenhum item fique "órfão".

───

4. MODULAÇÃO DOS MENUS DE EXPORTAÇÃO E IMPORTAÇÃO
Há três fluxos completamente separados para não confundir o usuário. A IA deve manter essa separação:

1. Importar / Exportar (A Base de Dados Oficial - JSON)
• Objetivo: Sincronizar o Fábio e a Bruna.
• O fluxo exige um botão proeminente de Salvar Agora na tela inicial. Ao clicar, o sistema salva localmente, carimba o horário na tela e pergunta se quer exportar o JSON atualizado e compartilhar (acionando a Web Share API do celular navigator.share).
• Ao abrir o site, um Pop-up de Inicialização (startupModal) interroga o usuário se ele tem um arquivo JSON recente da outra pessoa para injetar. Esse pop-up usa sessionStorage para não ficar aparecendo repetidamente caso a página recarregue após a importação.

2. Relatórios (Para visualização externa)
• Gera PDFs bonitos (jsPDF-AutoTable) ou planilhas CSV ultra-simples contendo apenas: Nome, Quantidade, Categoria, Estabelecimento e Preço.
• Formato limpo, feito para leitura ou impressão.

3. Área Administrativa (Lote CSV)
• Foco: Atualização maciça de dados usando Excel.
• Possui design e avisos padrão (não coloridos/chamativos) indicando que é uma área crítica.
• Zerar Base de Dados: Botão vermelho que exige confirmação dupla por prompt (digitar "ZERAR") para apagar os itens.
• Upload CSV: Antes de abrir o seletor de arquivos, aciona um alerta perguntando se o usuário já limpou a base para não duplicar itens acidentalmente.
• Atenção à formatação estrita do CSV Administrador:Nome, Qtd, Unidade, Categoria, Subcategoria, AlertaVermelho, AlertaLaranja, Validade, Estabelecimento, Preco, Tags. Tags separadas por ponto-e-vírgula (;).

───

5. LISTA DE COMPRAS 2.0 (O Motor de Automação)
• Em vez de exigir que o usuário crie a lista manualmente, o sistema varre todos os itens. Aqueles que retornam status Crítico (Vermelho) ou Atenção (Laranja) são marcados automaticamente (checked).
• A janela possui sua própria barra de pesquisa e seus próprios botões de filtro por Categoria, Cor e Estabelecimento, operando de forma independente da tela principal.
• Uma Checkbox permite decidir se o texto copiado irá com a quantidade atual escrita na frente: [ ] Produto (Tem: X) ou apenas [ ] Produto.
• A função de copiar usa document.execCommand('copy') sobre um textarea oculto, pois o navigator.clipboard é instável no ecossistema mobile.

───

6. INSTRUÇÕES PARA CONTINUIDADE DO DESENVOLVIMENTO (Para a IA)

Ao gerar novo código ou sugerir melhorias para este projeto, você deve:

1. Reverenciar os Identificadores Atuais: Respeite rigorosamente as IDs do HTML (#itemsTableBody, #itemModal, #startupModal, etc) e as chaves do objeto appData para não corromper o banco de dados armazenado nos celulares dos usuários.
2. Manutenção do CSS Tailwind: Sempre use a sintaxe de classes em linha. Para animações customizadas e o CSS específico do Drag & Drop (a classe .over), utilize a tag <style> fixada no <head>.
3. Zelar pela Performance: Operações de busca, filtro e ordenação (Sort) ocorrem em tempo real a cada caractere digitado (oninput). O código JS dessas funções deve permanecer performático.
4. Sem Regressões (No Regressions): Não remova botões, não misture os modais de exportação, não traga o sistema de GitHub de volta, não altere o esquema de nomenclatura. Adicione novas funcionalidades ao redor do código existente com cuidado cirúrgico.
5. Output de Respostas: Quando for fornecer atualizações substanciais do código, forneça sempre a estrutura do arquivo completo (<!DOCTYPE html&gt;...&lt;/html>), pois como é um WebApp de arquivo único, pedaços de código podem causar erros de montagem humana.

(Fim do Dossiê Técnico do Projeto EMOHOUSE).