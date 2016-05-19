# Messenger bundle
---

Messenger application based on the websocket server and for the most operations websocket requests used rather than AJAX request.
The  main goal of the websocket server is to notify the users subscribed to it when appear any messages related to this users.
Websocket client (in web interface) and server part is based on GosWebSocketBundle.
 
## API references
---
There are two main entry points in REST API that provides all available data to show base messenger layout:
> 1. [Private account base data](https://stage.payever.de/api/rest/v1/doc#get--api-rest-v1-messenger-private-)
> 2. [Business account base data](https://stage.payever.de/api/rest/v1/doc#get--api-rest-v1-messenger-business-{slug})

Once the WS URL is available a new websocket connection can established to run the messenger application.
For the third-party application to authorize with websocket server it is required to send the OAuth token header just like
it is done for the usual REST API request.
Header example:

```
Authorization: Bearer SOME-LONG-TOKEN-STRING
```

Websocket server use WAMP subprotocol. Pubsub approach is used for getting new messages notifications. RPC approach used for
sending messages, marketing groups manipulation and so on.

### Subscription to new messages notifications.
Is done via subscription call to websocket:

```
[
  5,
  "messenger/chat/channel/{userId}/"
]
```

Where `{userId}` is a messenger user ID that is abtained in a first REST API call to get base data.
 
Response example:

```
(
  8,
  "messenger/chat/channel/{userId}/",
  {
    debug = 1;
    msg = " has joined messenger/chat/channel/{userId}/";
    userId = {userId};
  }
)
```

In case of unauthorized access connection will be closed. This can happen if the OAuth token has expired
or if OAuth token related to a user that is not an owner of the specified messenger channel.
   
### Get conversation data

This one and further operations implements WAMP RPC calls. This method can be used to get data about conversation and its messages.
 
Request example:
 
```
 [
  2,
  {tempChannelName},
  "messenger/rpc/getConversation"
  {
    id: {conversationsId},
    userId: {userId}
    startId: {startId},
    limit: {limit},
  }
]
```
 
Where:
`{tempChannelName}` - is some hash to identify the response once it will come back
`{conversationsId}` - ID of the selecte conversation
`{userId}` - messenger user ID
`{startId}` - the earliest message ID that is shown in the chat
`{limit}` - optional messages limit number to get. By default it is equal to 30

Apllication can use the `{startId}` and `{limit}` parameters to get all the messages history.
  
Response example:
 
```
[
  3,
  {tempChannelName},
  {
    id: {conversationsId},
    name: {conversationName}
    type: 'conversation'
    messages: {
      [
        id: {messageId}
        body: {messageBody}
        date: {date}
        own: {ownFlag}
        senderName: {senderName}
        avatar: {
          type: {avatarType},
          value: {avatarValue}
        }
        conversationId: {conversationsId} 
        conversationName: {conversationName}
        conversationType: 'conversation'
      ],
      ...
    }
  }
]
```
 
Where
`{conversationName}` - name of the conversation to show in the contacts list
 {messageId}` - message unique ID
`{messageBody}` - HTML-view of the message
`{date}` - formated date
`{ownFlag}` - is message own for current user
`{senderName}` - name of the sender
`{avatarType}` - can be one of `url` and  `letters`
`{avatarValue}` - relative path to image in case of `url` type and first letters of the name in `letters` type case
 
### Activity feed

Get activity feed messages. Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/getActivity",
  {
    userId: {userId},
    startId: {startId},
    limit: {limit}
  }
```

Response example:

```
[
  3,
  {tempChannelName},
  {
    id: null,
    name: 'Activity',
    type: 'activity'
    messages: {
      ...
    }
  }
]
```

### Marketing group

Get marketing group activity messages. This list includes messages from group participants and group owner's messages
sent directly to group.

Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/getMarketingGroup",
  {
    userId: {userId},
    startId: {startId},
    limit: {limit}
  }
```

Response example:

```
[
  3,
  {tempChannelName},
  {
    id: null,
    name: {groupName},
    type: 'marketing-group'
    messages: {
      ...
    }
  }
]
```

### Send message

Send message to specified conversation by ID. Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/sendMessage",
  {
    userId: {userId},
    conversationId: {conversationId},
    body: {body}
  }
]
```

Where `{body}` can include HTML-tags `<br><p><b><i><u><h1><h2><h3><a>`.

Response example:

```
[
  3,
  {tempChannelName},
  {
    id: {messageId}
    body: {messageBody}
    date: {date}
    own: {ownFlag}
    senderName: {senderName}
    avatar: {
      type: {avatarType},
      value: {avatarValue}
    }
    conversationId: {conversationsId} 
    conversationName: {conversationName}
    conversationType: 'conversation'
  }
]
```

### Conversation settings

Get conversations data to show settings window. Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/getConversationSettings",
  {
    id: {conversationId},
    userId: {userId},
  }
]
```

Response example:
```
[
  3,
  {tempChannelName},
  {
    id: {conversationId}
    name: {conversationName}
    avatar: {
      type: {avatarType},
      value: {avatarValue}
    }
  }
]
```

Where
`{conversationName}` - is name of the second participant of chat and his avatar in respective key.


### Marketing group settings

Get marketing roup data to show settings window. Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/getMarketingGroupSettings",
  {
    id: {marketingGroupId},
    userId: {userId},
  }
]
```

Response example:
```
[
  3,
  {tempChannelName},
  {
    id: {marketingGroupId}
    name: {marketingGroupName}
    members: [
      {
        id: {memberId}
        name: {memberName}
        avatar: {
          type: {avatarType},
          value: {avatarValue}
        }
      },
      ...
    ]
  }
]
```

Where
`{memberId}` - is ID of marketing group member unique accross all groups
`{memberName}` - is name of the group member

### Add marketing group member

Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/addMarketingGroupMember",
  {
    groupId: {marketingGroupId},
    userId: {userId},
    memberAlias: {memberAlias}
  }
]
```

Where `{memberAlias}` has one of the following formats: 'user-{privateId}', 'business-{businessId}' and 'contact-{contactId}'

Response example:
```
[
  3,
  {tempChannelName},
  {
    id: {memberId}
    name: {memberName}
    avatar: {
      type: {avatarType},
      value: {avatarValue}
    }
  }
]
```

### Remove marketing group member

Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/removeMarketingGroupMember",
  {
    groupId: {marketingGroupId},
    memberId: {memberId},
    userId: {userId},
  }
]
```

Where `{memberId}` is ID of marketing group member unique accross all groups

Response example:
```
[
  3,
  {tempChannelName},
  {
    removed: true
  }
]
```

### Delete marketing group

Request example:

```
[
  2,
  {tempChannelName},
  "messenger/rpc/deleteMarketingGroup",
  {
    groupId: {marketingGroupId},
    userId: {userId},
  }
]
```

Response example:
```
[
  3,
  {tempChannelName},
  {
    deleted: true
  }
]
```
