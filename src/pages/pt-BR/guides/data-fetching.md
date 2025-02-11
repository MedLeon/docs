---
layout: ~/layouts/MainLayout.astro
title: Busca de Dados
description: Aprenda como buscar dados remotamente com Astro utilizando a API fetch.
i18nReady: true
---

Arquivos `.astro` podem buscar dados remotamente em tempo de construção para te ajudar a gerar suas páginas.

## `fetch()` em Astro

Todos os [componentes Astro](/pt-BR/core-concepts/astro-components/) tem acesso a [função global `fetch()`](https://developer.mozilla.org/pt-BR/docs/Web/API/fetch) em seus scripts do componente para fazer requisições HTTP a APIs. Essa chamada ao `fetch` será executada em tempo de construção, e os dados estarão disponíveis ao template do componente para gerar HTML dinâmico.

💡 Se aproveite de [**top-level await**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await#top_level_await) dentro do script do seu componente Astro.

💡 Passe os dados buscados para componentes Astro e de outros frameworks como props.

```astro
---
// src/components/Usuario.astro
import Contato from '../components/Contato.jsx';
import Localizacao from '../components/Localizacao.astro';

const resposta = await fetch('https://randomuser.me/api/');
const dados = await resposta.json();
const usuarioAleatorio = dados.results[0]
---
<!-- Dados buscados em tempo de construção podem ser renderizados no HTML -->
<h1>Usuário</h1>
<h2>{usuarioAleatorio.name.first} {usuarioAleatorio.name.last}</h2>

<!-- Dados buscados em tempo de construção podem ser passados aos componentes como props -->
<Contato client:load email={usuarioAleatorio.email} />
<Localizacao city={usuarioAleatorio.location.city} />
```

### Consultas GraphQL

Astro também pode utilizar `fetch()` para consultar um servidor GraphQL com qualquer consulta GraphQL válida.

```astro
---
const resposta = await fetch("https://graphql-weather-api.herokuapp.com",
  {
    method:'POST',
    headers: {'Content-Type':'application/json'},
    body: JSON.stringify({
      query: `
        query getWeather($name:String!) {
            getCityByName(name: $name){
              name
              country
              weather {
                summary {
                    description
                }
              }
            }
          }
        `,
      variables: {
          name: "Toronto",
      },
    }),
  })

const json = await resposta.json();
const clima = json.data
---
<h1>Buscando o clima em tempo de construção</h1>
<h2>{clima.getCityByName.name}, {clima.getCityByName.country}</h2>
<p>Clima: {clima.getCityByName.clima.summary.description}</p>
```

> 💡 Lembre-se, todos os dados em componentes Astro são buscados quando o componente é renderizado.

Seu site Astro após o deploy irá buscar os dados **uma vez, em tempo de construção**. No desenvolvimento, você verá a busca de dados ao recarregar componentes. Se você precisa buscar dados múltiplas vezes no lado do cliente, utilize um [componente de framework](/pt-BR/core-concepts/framework-components/) ou um [script no lado do cliente](/pt-BR/core-concepts/astro-components/#scripts-no-lado-do-cliente) em um componente Astro.

## `fetch()` em Componentes de Frameworks

A função `fetch()` também está globalmente disponível a qualquer [componente de framework](/pt-BR/core-concepts/framework-components/):

```tsx
// Filmes.tsx
import type { FunctionalComponent } from 'preact';
import { h } from 'preact';

const dados = await fetch('https://exemplo.com/filmes.json').then((resposta) =>
  resposta.json()
);

// Componentes que são renderizados no momento de construção também fazem logs na interface de linha de comando.
// Quando renderizado com uma diretiva client:*, eles também irão fazer logs no console do navegador.
console.log(dados);

const Filmes: FunctionalComponent = () => {
// Exibe o resultado na página
  return <div>{JSON.stringify(dados)}</div>;
};

export default Filmes;
```
