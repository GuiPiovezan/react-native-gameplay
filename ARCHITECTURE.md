# Arquitetura do Projeto GamePlay

## Visão Geral

O **GamePlay** é uma aplicação móvel desenvolvida em **React Native** com **TypeScript** e **Expo**, projetada para facilitar a organização de partidas online entre jogadores. A aplicação utiliza autenticação OAuth2 do Discord e permite aos usuários agendar partidas, gerenciar servidores (guilds) e visualizar informações de seus grupos de jogos.

## Stack Tecnológica

### Core
- **React Native** (SDK 41): Framework para desenvolvimento mobile multiplataforma
- **TypeScript**: Superset JavaScript para tipagem estática
- **Expo**: Plataforma para desenvolvimento e deploy de apps React Native

### Navegação
- **React Navigation v5**: Biblioteca de navegação para React Native
  - Stack Navigator para navegação entre telas

### Gerenciamento de Estado
- **React Context API**: Para gerenciamento de estado global de autenticação
- **React Hooks**: useState, useEffect, useCallback, useContext para gerenciamento de estado local

### Armazenamento
- **AsyncStorage**: Persistência local de dados (usuários e agendamentos)

### Autenticação
- **Expo Auth Session**: Fluxo OAuth2 para autenticação com Discord
- **Discord API**: Integração com API do Discord para autenticação e obtenção de dados de guilds

### HTTP Client
- **Axios**: Cliente HTTP para comunicação com APIs externas

### UI/UX
- **Expo Linear Gradient**: Gradientes para componentes visuais
- **React Native Vector Icons**: Ícones personalizados
- **React Native SVG**: Suporte a arquivos SVG
- **React Native Gesture Handler**: Gestos e interações avançadas
- **React Native Reanimated**: Animações de alta performance

### Fontes
- **@expo-google-fonts/inter**: Família de fontes Inter
- **@expo-google-fonts/rajdhani**: Família de fontes Rajdhani

## Estrutura de Diretórios

```
react-native-gameplay/
├── src/
│   ├── @types/              # Declarações de tipos TypeScript
│   │   ├── png.d.ts         # Tipos para arquivos PNG
│   │   └── svg.d.ts         # Tipos para arquivos SVG
│   │
│   ├── assets/              # Recursos estáticos (imagens, SVGs)
│   │
│   ├── components/          # Componentes reutilizáveis
│   │   ├── Appointments/    # Card de agendamento
│   │   ├── Avatar/          # Avatar do usuário
│   │   ├── Background/      # Componente de fundo com gradiente
│   │   ├── Button/          # Botão principal
│   │   ├── ButtonAdd/       # Botão de adicionar
│   │   ├── ButtonIcon/      # Botão com ícone
│   │   ├── Category/        # Card de categoria
│   │   ├── CategorySelect/  # Seletor de categorias
│   │   ├── Guild/           # Card de guild/servidor
│   │   ├── GuildIcon/       # Ícone de guild
│   │   ├── Header/          # Cabeçalho de páginas
│   │   ├── ListDivider/     # Divisor de lista
│   │   ├── ListHeader/      # Cabeçalho de lista
│   │   ├── Load/            # Indicador de carregamento
│   │   ├── Member/          # Card de membro
│   │   ├── ModalView/       # Modal customizado
│   │   ├── Profile/         # Perfil do usuário
│   │   ├── SmallInput/      # Input pequeno
│   │   └── TextArea/        # Área de texto
│   │
│   ├── configs/             # Configurações da aplicação
│   │   └── database.ts      # Constantes para AsyncStorage
│   │
│   ├── global/              # Estilos e configurações globais
│   │   └── styles/
│   │       └── theme.ts     # Tema com cores e fontes
│   │
│   ├── hooks/               # Custom Hooks
│   │   └── auth.tsx         # Hook de autenticação
│   │
│   ├── routes/              # Configuração de rotas
│   │   ├── index.tsx        # Router principal
│   │   └── app.routes.tsx   # Rotas da aplicação
│   │
│   ├── screens/             # Telas da aplicação
│   │   ├── SiginScreen/     # Tela de login
│   │   ├── HomeScreen/      # Tela inicial
│   │   ├── AppointmentCreate/ # Tela de criar agendamento
│   │   ├── AppointmentDetails/ # Tela de detalhes do agendamento
│   │   └── Guilds/          # Tela de seleção de guilds
│   │
│   ├── services/            # Serviços e APIs
│   │   └── api.ts           # Configuração do cliente Axios
│   │
│   └── utils/               # Utilitários
│       └── categories.ts    # Definição de categorias de jogos
│
├── App.tsx                  # Componente raiz da aplicação
├── app.json                 # Configuração do Expo
├── package.json             # Dependências do projeto
├── tsconfig.json            # Configuração do TypeScript
└── babel.config.js          # Configuração do Babel
```

## Arquitetura de Componentes

### Padrão de Organização

Cada componente segue a estrutura:
```
ComponentName/
├── index.tsx    # Lógica e estrutura do componente
└── styles.ts    # Estilos usando StyleSheet
```

### Componentização

A aplicação segue o princípio de componentização, dividindo a interface em componentes reutilizáveis e independentes:

- **Componentes de Layout**: Background, Header, ModalView
- **Componentes de Formulário**: Button, SmallInput, TextArea, CategorySelect
- **Componentes de Listagem**: ListHeader, ListDivider, Guild, Appointments, Member
- **Componentes de UI**: Avatar, Profile, Load, GuildIcon

## Fluxo de Autenticação

### OAuth2 com Discord

```
┌─────────────┐
│   Usuário   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│   SigInScreen (Tela de Login)  │
└────────────┬────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│   AuthContext.signIn()          │
│   - Inicia fluxo OAuth2         │
│   - Redireciona para Discord    │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│   Discord OAuth2                 │
│   - Usuário autoriza app        │
│   - Retorna access_token         │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│   Obter dados do usuário         │
│   - GET /users/@me               │
│   - Salva no AsyncStorage        │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│   HomeScreen                     │
│   (Usuário autenticado)          │
└──────────────────────────────────┘
```

### Persistência de Sessão

1. Ao fazer login, os dados do usuário são salvos no `AsyncStorage`
2. Na inicialização, o app verifica se há dados salvos
3. Se houver, restaura a sessão automaticamente
4. O token é adicionado aos headers da API

## Fluxo de Navegação

```
                    ┌────────────────┐
                    │  Routes        │
                    │  (index.tsx)   │
                    └───────┬────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
        ┌──────────────┐        ┌──────────────┐
        │ !user.id     │        │  user.id     │
        │ SigInScreen  │        │  AppRoutes   │
        └──────────────┘        └──────┬───────┘
                                       │
                        ┌──────────────┼──────────────┐
                        │              │              │
                        ▼              ▼              ▼
                ┌────────────┐ ┌────────────┐ ┌────────────┐
                │ HomeScreen │ │Appointment │ │Appointment │
                │            │ │  Details   │ │  Create    │
                └────────────┘ └────────────┘ └────────────┘
                                                     │
                                                     ▼
                                              ┌────────────┐
                                              │   Guilds   │
                                              │  (Modal)   │
                                              └────────────┘
```

## Gerenciamento de Dados

### AsyncStorage - Coleções

```typescript
@gameplay:user          // Dados do usuário autenticado
@gameplay:appointments  // Lista de agendamentos criados
```

### Estrutura de Dados

#### User
```typescript
{
  id: string;
  userName: string;
  firtsName: string;
  avatar: string;
  email: string;
  token: string;
}
```

#### Appointment
```typescript
{
  id: string;
  guild: GuildProps;
  category: string;
  date: string;
  description: string;
}
```

#### Guild
```typescript
{
  id: string;
  name: string;
  icon: string;
  owner: boolean;
}
```

## Integração com Discord API

### Endpoints Utilizados

1. **Autenticação OAuth2**
   ```
   GET /oauth2/authorize
   - client_id
   - redirect_uri
   - response_type
   - scope
   ```

2. **Obter Dados do Usuário**
   ```
   GET /users/@me
   Authorization: Bearer {token}
   ```

3. **Listar Guilds do Usuário**
   ```
   GET /users/@me/guilds
   Authorization: Bearer {token}
   ```

### Configuração da API

```typescript
// services/api.ts
const api = axios.create({
  baseURL: 'https://discord.com/api'
});
```

Os tokens são configurados dinamicamente após autenticação:
```typescript
api.defaults.headers.authorization = `Bearer ${access_token}`;
```

## Sistema de Categorias

As partidas são organizadas em 4 categorias:

1. **Ranqueada** - Partidas competitivas ranqueadas
2. **Duelo 1x1** - Partidas individuais
3. **Diversão** - Partidas casuais
4. **Treino** - Sessões de treino

Cada categoria possui:
- ID único
- Título
- Ícone SVG

## Tema e Estilização

### Paleta de Cores

```typescript
colors: {
  primary: '#E51C44',           // Cor principal (vermelho)
  secondary100: '#0A1033',      // Fundo mais escuro
  secondary90: '#0D133D',
  secondary80: '#0E1647',
  secondary70: '#1B2565',
  secondary50: '#243189',
  secondary40: '#1D2766',
  secondary30: '#495BCC',
  overlay: 'rgba(0,0,0,0.7)',
  highlight: '#ABB1CC',         // Textos secundários
  heading: '#DDE3F0',           // Títulos
  line: '#991F36',              // Linhas divisórias
  on: '#32BD50',                // Status online
  discord: '#7289da'            // Cor do Discord
}
```

### Tipografia

```typescript
fonts: {
  title700: 'Rajdhani_700Bold',    // Títulos em negrito
  title500: 'Rajdhani_500Medium',  // Títulos médios
  text400: 'Inter_400Regular',     // Texto regular
  text500: 'Inter_500Medium'       // Texto médio
}
```

## Funcionalidades Principais

### 1. Autenticação
- Login via Discord OAuth2
- Persistência de sessão
- Logout com limpeza de dados

### 2. Visualização de Agendamentos
- Lista de partidas agendadas
- Filtro por categoria
- Contador de partidas

### 3. Criação de Agendamentos
- Seleção de categoria
- Escolha de servidor (guild)
- Definição de data e hora
- Descrição da partida

### 4. Detalhes do Agendamento
- Informações completas da partida
- Lista de membros do servidor
- Opção de compartilhamento

### 5. Gerenciamento de Guilds
- Listagem de servidores do Discord
- Seleção para agendamento
- Exibição de ícones dos servidores

## Padrões de Desenvolvimento

### Hooks Customizados

**useAuth**: Centraliza toda lógica de autenticação
```typescript
const { user, loading, signIn, signOut } = useAuth();
```

### Styled Components Pattern

Separação clara entre lógica e apresentação:
- `index.tsx`: Lógica do componente
- `styles.ts`: Estilos usando StyleSheet

### Componentes Controlados

Formulários utilizam estado local para controle de inputs:
```typescript
const [day, setDay] = useState('');
const [month, setMonth] = useState('');
```

### Navigation Props

Tipagem forte para navegação entre telas:
```typescript
navigation.navigate('AppointmentDetails', { guildSelected });
```

## Segurança

### Variáveis de Ambiente

Informações sensíveis são armazenadas em `.env`:
- `CLIENT_ID`: ID da aplicação Discord
- `REDIRECT_URI`: URI de redirecionamento OAuth
- `SCOPE`: Escopos de permissões
- `CDN_IMAGE`: URL do CDN do Discord
- `RESPONSE_TYPE`: Tipo de resposta OAuth

### Tokens

- Tokens são armazenados localmente no AsyncStorage
- Tokens são incluídos em todos os requests autenticados
- Tokens são removidos no logout

## Melhorias Futuras

### Possíveis Implementações

1. **Backend Próprio**
   - Banco de dados para agendamentos
   - API REST para sincronização
   - Notificações push

2. **Funcionalidades Adicionais**
   - Chat em tempo real
   - Sistema de notificações
   - Calendário de partidas
   - Histórico de partidas
   - Sistema de ranking

3. **Otimizações**
   - Cache de dados
   - Paginação de listas
   - Lazy loading de componentes
   - Otimização de imagens

4. **Testes**
   - Testes unitários com Jest
   - Testes de integração
   - Testes E2E com Detox

## Conclusão

O **GamePlay** é uma aplicação bem estruturada que segue as melhores práticas de desenvolvimento React Native. A arquitetura modular facilita manutenção e escalabilidade, enquanto a integração com Discord API proporciona uma experiência fluida para os usuários organizarem suas partidas online.

A separação clara de responsabilidades, o uso de TypeScript para tipagem estática e a componentização adequada tornam o código limpo, legível e fácil de manter.
