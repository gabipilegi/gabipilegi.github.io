## Crud simples com Ring e Compojure

## O que?

O que nossa aplicação web faz
Processa requests e gera respostas

Operações CRUD via HTTP

## Como?

### Ring

Biblioteca de aplicações web mais utilizada no clojure.

Utilizaremos para construir o nosso serviço web.

Consiste de 4 componentes:

- Requests
    
    Mapa de entrada
    
- Responses
    
    Mapa de saída
    
- Handlers
    
    Função que recebe um único argumento (Mapa da Request)
    
    e devolve uma única saída (Mapa da Saída)
    
- Middleware

Request Map → Handler → Response Map

### Compojure

 Biblioteca compacta para roteamento

### Jetty

Como servidor. 
Software que permite a interação com o servidor através da rede

REST: Representation State Transfer

## Hello World com Ring e Jetty

1. Adicionar as dependências do `ring/ring-core`  e do `ring/ring-jetty-adapter` em um arquivo deps.edn

1. Define um handler que sempre retorna 200 para qualquer request

```clojure
(defn handler [request] 
	{:status 200 :body "Hello World!"})
```

1. Inicia o Jetty web server, passando o handler e opções

```clojure
(require '[ring.adapter.jetty :refer [run-jetty]])

(def app (run-jetty handler 
										{:port 8080
                     :join? false}))
```

> `:join?` opção para bloquear o REPL ao rodar o servidor
> 

Para parar o servidor 

```clojure
(.stop app)
```

### Roteamento de requisições

Queremos aceitar requests para múltiplas rotas e métodos

O objeto de request possui:

- URI
- Request-Method

Para isso utilizaremos o **Compojure**

### Utilizando Compojure

Adicionar a dependência do `compojure/compojure`

```clojure
(require '[compojure.route :as route])
(require '[compojure.core :refer :all])
```

```clojure
(def route 
	(GET "/hello" request "Hello World!"))
```

- `GET`: Macro que define uma rota do método GET
- `"/Hello"`: Rota
- `request`: binding da request em símbolos para usar na função da rota
- `Hello World`: O corpo da rota, onde adicionaremos a lógica

```clojure
(defroutes routes 
  (GET "/hello" request "Hello!")
	(GET "/hello" request "Hello 2")
  ;; --- demais rotas
)
```

Ao buscar uma rota desconhecida tomamos um erro, podemos adicionar uma rota padrão para redirecionar quando não t

```clojure
(require '[compojure.route :as route])

(defroutes routes 
  (GET "/hello" request "Hello!")
	(GET "/hello" request "Hello 2")
  (route/not-found "Rota não encontrada")
)
```

💡 Se atente para deixar a rota de "*Not found*" por último 

## Middleware

Permite programar tratamentos genéricos que deveriam ser aplicados em todas as requisições

```clojure
(defn custom-middleware
	[handler]
	(fn [request]
			(->> request 
           ;; manipula a request
						handler
						;; manipula a resposta)))
```

Pode tratar a request antes de chamar o handler e/ou a resposta após a execução do handler.

### Formatos de respostas

Múltiplos tipos de cliente podem aceitar diferentes formatos de resposta, um frontend JS prefere enviar e receber payloads em JSON enquanto aplicações em Clojure Script vão preferir EDN.

```clojure
;;EDN
{:chave-exemplo "valor exemplo"}
;;JSON
{"chaveExemplo": "valor exemplo"}
```

**Muuntaja**: Ferramenta para fazer negociação de conteúdo e formatação de requisição e resposta, permite lidar com as exigências de formatos dos clientes mantendo a aplicação agnóstica a isso.

1. Adicionar a dependência do `metosin/muuntaja`

1. Adicionar referência do muuntaja

```clojure
(require '[muuntaja.middleware :as middleware])
```

1. Adicionar uma rota complexa

```clojure
(require '[compojure.core :refer :all])

(defroutes my-routes
	(GET "/estruturada" request 
		{:body {:a 1
						:b #{1 2}
            :c "outra coisa"}}))
```

1. rodar o app

```clojure
(require '[ring.adapter.jetty :refer [run-jetty]])

(run-jetty (middleware/wrap-format my-routes) {:port 8080 :join? false})
```

```bash
$ curl -i http://localhost:8080/estruturada
```

```bash
$ curl -i http://localhost:8080/estruturada -H "accept: application/edn"
```

## CRUD

Emacs:

```clojure
(server
  (:require [compojure.core :refer [defroutes DELETE GET PUT]]
            [compojure.route :as route]))

(def db (atom {}))

(defroutes service-routes
  (GET "/entidade" request
       {:body
        (when-let [entidade (@db :data)])})
  (PUT "/entidade" request
       (swap! db assoc :data (:body-params request))
       {:status 201})
  (DELETE "/enntidade" request
          (swap! db dissoc :data))
  (route/not-found "Not found"))
```
