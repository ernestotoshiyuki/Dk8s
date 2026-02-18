# Criando imagens de containers e reduzindo vulnerabilidades com Wolfi
Aplicaçao giropops senhas do repositório https://github.com/badtuxx/giropops-senhas

buildar a imagem que está no Dockerfile

        build -t giropops-senahs:1.0 .

Verifique se a imagem foi criada com
        
        docker image ls

Execute a imagem para validar
        
        docker run -d --name giropops-senhas -p 5000:5000 giropops-senhas:1.0

Verifique se subiu
        
        docker ps

Verifique os logs
        
        docker container logs giropops-senhas
Note que irá dar um Internal server erro por precisar ainda do Redis

Suba um redis com
        
        docker run -d --name redis -p 6379:6379 redis

Teste a conexao com um curl no IP:6379

Mate e suba novamente o container giropops para dar um export do Redis
        
        docker rm -f giropops-senhas
        docker run -d --name giropops-senhas -p 5000:5000 -e REDIS_HOST=<IP> giropops-senhas:1.0

# A Arte da Minimização: Imagens Distroless

## Introdução

No mundo dos containers, existe uma tendência emergente chamada "Distroless". Este termo é um neologismo que descreve uma imagem de container que contém apenas as dependências e artefatos necessários para a execução do aplicativo, eliminando a distribuição completa do sistema operacional. Neste capítulo, exploraremos o conceito de distroless, suas vantagens, desvantagens e como implementá-lo em seu próprio ambiente.

## O que é Distroless?
 
"Distroless" refere-se à estratégia de construir imagens de containers que são efetivamente "sem distribuição". Isso significa que elas não contêm a distribuição típica de um sistema operacional Linux, como shell de linha de comando, utilitários ou bibliotecas desnecessárias. Ao invés disso, essas imagens contêm apenas o ambiente de runtime (como Node.js, Python, Java, etc.) e o aplicativo em si.

## Benefícios do Distroless

O principal benefício das imagens Distroless é a segurança. Ao excluir componentes não necessários, o potencial de ataque é significativamente reduzido. Além disso, sem as partes extras do sistema operacional, o tamanho do container é minimizado, economizando espaço de armazenamento e aumentando a velocidade de deploy.

## Desafios do Distroless

Apesar de seus benefícios, a estratégia Distroless não está isenta de desafios. Sem um shell ou utilitários de sistema operacional, a depuração pode ser mais complicada. Além disso, a construção de imagens Distroless pode ser um pouco mais complexa, pois requer uma compreensão cuidadosa das dependências do aplicativo.

## Implementando Distroless

Existem várias maneiras de implementar uma estratégia Distroless. A mais simples é usar uma das imagens base Distroless fornecidas pelo Google e pela Chainguard. Estas imagens são projetadas para serem o mais minimalistas possível e podem ser usadas como ponto de partida para a construção de suas próprias imagens de container.

## Conclusão

Distroless representa uma evolução na maneira como pensamos sobre imagens de containers. Ele nos força a questionar o que realmente precisamos em nosso ambiente de execução e nos encoraja a minimizar o excesso. Embora a implementação de uma estratégia Distroless possa ser um desafio, os benefícios em termos de segurança e eficiência tornam essa uma consideração valiosa para qualquer equipe de desenvolvimento.

# Docker Scout 

À medida que o desenvolvimento de software se torna cada vez mais orientado para containers, a segurança desses containers e das imagens usadas para criá-los ganha importância crítica. As imagens de container são construídas a partir de camadas de outras imagens e pacotes de software. Infelizmente, essas camadas e pacotes podem conter vulnerabilidades que tornam os containers e os aplicativos que eles executam vulneráveis a ataques. Aqui é onde o Docker Scout entra em cena.

## O que é o Docker Scout?

O Docker Scout é uma ferramenta de análise de imagens avançada oferecida pelo Docker. Ele foi projetado para ajudar desenvolvedores e equipes de operações a identificar e corrigir vulnerabilidades em suas imagens de containers. Ao analisar suas imagens, o Docker Scout cria um inventário completo dos pacotes e camadas, também conhecido como Software Bill of Materials (SBOM). Este inventário é então correlacionado com um banco de dados de vulnerabilidades atualizado continuamente para identificar possíveis problemas de segurança.

## Como o Docker Scout funciona?

Você pode usar o Docker Scout de várias maneiras. Ele é integrado ao Docker Desktop e ao Docker Hub, facilitando a análise de imagens durante o processo de construção e implantação. Além disso, ele também pode ser usado em um pipeline de integração contínua, através da interface de linha de comando (CLI) do Docker e no Docker Scout Dashboard.

Para aqueles que hospedam suas imagens no JFrog Artifactory, o Docker Scout também oferece suporte à análise de imagens nesse ambiente.

No CLI do Docker, o Docker Scout oferece vários comandos, incluindo compare para comparar duas imagens, cves para exibir as vulnerabilidades conhecidas como CVEs identificadas em um artefato de software, quickview para uma visão geral rápida de uma imagem e recommendations para exibir atualizações de imagens base disponíveis e recomendações de correção.

## Usando o Docker Scout

O comando docker scout cves é especialmente importante, pois permite analisar um artefato de software em busca de vulnerabilidades. Este comando suporta a análise de imagens, diretórios OCI layout e arquivos tarball, como os criados pelo comando docker save. Isso dá aos desenvolvedores a flexibilidade de verificar a segurança de suas imagens de container de várias maneiras.

## Por que o Docker Scout é importante?

O Docker Scout é uma ferramenta valiosa para melhorar a segurança dos containers. Ao identificar proativamente as vulnerabilidades e fornecer correções recomendadas, ajuda as equipes de desenvolvimento a fortalecer suas imagens de containers e, por sua vez, os aplicativos que estão sendo executados nesses containers. Em um mundo onde a segurança do software é fundamental, o Docker Scout é uma ferramenta indispensável para qualquer equipe de desenvolvimento orientada a containers.

# Instalando o Cosing
```bash
curl -O -L "https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64"
sudo mv cosign-linux-amd64 /usr/local/bin/cosign
sudo chmod +x /usr/local/bin/cosign
```
para gerar a chave

                cosign generate-key-pair

para fazer a assinatura da imagem ela tem que estar no docker hub
                 cosign sign --key cosign.key $IMAGE

para outra pessoa verificar a imagem assinada
                cosign verify --key cosing.pub $IMAGE