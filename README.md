<div align="center">
  <br />
  <p>
    <a href="https://discord.js.org"><img src="https://discord.js.org/static/logo.svg" width="546" alt="discord.js" /></a>
  </p>
  <br />
  <p>
    <a href="https://discord.gg/WMMWKhP"><img src="https://discordapp.com/api/guilds/222078108977594368/embed.png" alt="Discord server" /></a>
    <a href="https://www.npmjs.com/package/discord.js"><img src="https://img.shields.io/npm/v/discord.js.svg?maxAge=3600" alt="NPM version" /></a>
    <a href="https://www.npmjs.com/package/discord.js"><img src="https://img.shields.io/npm/dt/discord.js.svg?maxAge=3600" alt="NPM downloads" /></a>
    <a href="https://travis-ci.org/discordjs/discord.js"><img src="https://travis-ci.org/discordjs/discord.js.svg" alt="Build status" /></a>
    <a href="https://david-dm.org/discordjs/discord.js"><img src="https://img.shields.io/david/discordjs/discord.js.svg?maxAge=3600" alt="Dependencies" /></a>
    <a href="https://www.patreon.com/discordjs"><img src="https://img.shields.io/badge/donate-patreon-F96854.svg" alt="Patreon" /></a>
  </p>
  <p>
    <a href="https://nodei.co/npm/discord.js/"><img src="https://nodei.co/npm/discord.js.png?downloads=true&stars=true" alt="npm installnfo" /></a>
  </p>
</div>

# Perguntas frequentes


Nesta página, algumas perguntas muito básicas e frequentes são respondidas. É importante entender que **esses exemplos são genéricos** e provavelmente não funcionarão se você simplesmente copiá-los / colá-los em seu código. Você precisa **entender** essas linhas, e não apenas colocá-las cegamente em seu código.

## Exemplos de código

## Cliente Bot e Bot

```javascript
// Defina o status "Playing:" do bot (deve estar em um evento!)
// NOTA: ESTE MÉTODO É DEPRECADO A PARTIR DA VERSÃO 11.3 E SERÁ REMOVIDO NA VERSÃO 12
client.on("ready", () => {
    client.user.setGame("texto");
});

// NOTA: INTRODUZIDO NA VERSÃO 11.3 E SUBSTITUI setGame
client.on("ready", () => {
    client.user.setActivity({game: {name: "texto", type: 0}});
});



// Definir o bot online/idle/dnd/invisible status
client.on("ready", () => {
    client.user.setStatus("online");
});
```

## Usuários e membros

> Nestes exemplos, `Guild` é um espaço reservado para onde você recebe a guilda. Isso pode ser `message.guild` ou `member.guild` ou apenas `guild` dependendo do evento. Ou você pode obter a guilda por ID \ (veja a próxima seção \) e usar isso também!

```javascript
// Get a User by ID
client.users.get("user id");
// Returns <User>
```

```javascript
// Obter um membro por ID
message.guild.members.get("user ID");
// Returns <Member>
```

```javascript
// Obter um membro da menção de mensagem
message.mentions.members.first();
// Returns <Member>
```

```javascript
// Enviar uma mensagem direta para um usuário
message.author.send("hello");
// Com Member, também funciona:
message.member.send("Heya!");
```

```javascript
// Mencione um usuário em uma mensagem
message.channel.send(`Hello ${user}, and welcome!`);
// ou
message.channel.send("Hello " + message.author.toString() + ", and welcome!");
```

```javascript
// Restringir um comando a um usuário específico por ID
if (message.content.startsWith(prefix + 'commandname')) {
    if (message.author.id !== 'A user ID') return;
    // Seu comando aqui
}
```

```javascript
// BUSQUE um membro. Útil se um usuário invisível enviar uma mensagem.
message.guild.fetchMember(message.author)
  .then(member => {
    // O membro está disponível aqui.
  });

// ESTE MUDANÇAS NA VERSÃO DE DISCORD 12 !!!!!
message.guild.members.fetch(message.author)
  .then(member => {
    // O membro está disponível aqui.
  });
```

## Canais e Guildas

```javascript
// Get a Guild by ID
client.guilds.get("the guild id");
// Returns <Guild>
```

```javascript
// Get a Channel by ID
client.channels.get("the channel id");
// Returns <Channel>
```

```javascript
// Get a Channel by Name
message.guild.channels.find("name", "channel-name");
// returns <Channel>
```

```javascript
// Create an invite and send it in the channel
// You can only create an invite from a GuildChannel
// Messages can only be sent to a TextChannel
message.guild.channels.get('<CHANNEL ID>').createInvite().then(invite =>
    message.channel.send(invite.url)
);
```

### Canal padrão

A partir de 03/08/2017, ** não há mais Canal Padrão ** nas guildas da Discord. O canal padrão # # geral pode ser deletado, e a propriedade `guild.defaultChannel` não funciona mais. Como alternativa, para aqueles que realmente querem enviar para o que se parece com o canal padrão, aqui está uma solução suja.

```javascript
const getDefaultChannel = async (guild) => {
  // get "original" default channel
  if(guild.channel.has(guild.id))
    return guild.channels.get(guild.id)

  // Check for a "general" channel, which is often default chat
  if(guild.channels.exists("name", "general"))
    return guild.channels.find("name", "general");
  // Now we get into the heavy stuff: first channel in order where the bot can speak
  // hold on to your hats!
  return guild.channels
   .filter(c => c.type === "text" &&
     c.permissionsFor(guild.client.user).has("SEND_MESSAGES"))
   .sort((a, b) => a.position - b.position ||
     Long.fromString(a.id).sub(Long.fromString(b.id)).toNumber())
   .first();
}

// This is called as, for instance:
client.on("guildMemberAdd", member => {
  const channel = getDefaultChannel(member.guild);
  channel.send(`Welcome ${member} to the server, wooh!`);
});
```


É muito importante notar que, se o bot tiver admin perms, seu "Primeiro canal gravável" é o que está no topo. Isso poderia ser Regras, Anúncios, FAQs, o que for. Então, se o canal padrão foi excluído e não há um canal geral, você vai incomodar muita gente.

Considere usar [Enmap](https://npmjs.com/package/enmap) para configurações por guilda \([example here](https://gist.github.com/eslachance/5c539ccebde9fa76340fb5d54889aa22)\) e deixe os administradores do servidor **escolher** um canal!

## Messages

```javascript
// Editing a message the bot sent
message.channel.send("Test").then(sentMessage => sentMessage.edit("Blah"));
// message now reads : "Blah"
```

```javascript
// Fetching a message by ID (Discord.js versions 9 through 11)
// note: you can line return right before a "dot" in JS, that is valid.
message.channel.fetchMessages({around: "352292052538753025", limit: 1})
  .then(messages => {
    const fetchedMsg = messages.first(); // messages is a collection!)
    // do something with it
    fetchedMsg.edit("This fetched message was edited");
  });

// THIS CHANGES IN DISCORD VERSION 12!!!!!
message.channel.messages.fetch({around: "352292052538753025", limit: 1})
  .then(messages => {
    messages.first().edit("This fetched message was edited");
  });
```

