---
id: quick-reference
title: Quick Reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';

export const TransactionSnippet = ({variableName}) => <CodeBlock language="tsx">{`  const keyring = (new Keyring({ type: "sr25519" }))
  const senderKeyPair = keyring.addFromMnemonic('<your_mnemonic_here>')
  
  ${variableName}.signAndSend(senderKeyPair, async (result) => \{
      const { status } = result
      
      if(!result || !status){
        return;
      }
      if (status.isFinalized || status.isInBlock) {
        const blockHash = status.isFinalized
          ? status.asFinalized
          : status.asInBlock; 
          console.log('✅ Tx finalized. Block hash', blockHash.toString());
      } else if (result.isError) {
        console.log(JSON.stringify(result));
      } else {
        console.log('⏱ Current tx status:', status.type);
      }
  \})
`}</CodeBlock>

:::info 
Here is a collection of the most commonly used methods within Subsocial SDK. For more in-depth look into this library, please reference the [TypeDocs](https://docs.subsocial.network/js-docs/js-sdk/index.html).
:::

## Setup

### Install

```
  yarn add @subsocial/api 
```

Add type definitions and utils library:

```
  yarn add @subsocial/types @subsocial/definitions @subsocial/utils
```

### Import

```typescript
  import { newFlatSubsocialApi } from '@subsocial/api'
```

### Configuration

<Tabs
  defaultValue="testnet"
  values={[
    {label: 'TestNet', value: 'testnet'},
    {label: 'MainNet', value: 'mainnet'},
    {label: 'LocalNet', value: 'localnet'},
  ]}>
  <TabItem value="mainnet">

```ts
  const config = {
    substrateNodeUrl: 'wss://rpc.subsocial.network',
    offchainUrl: 'https://app.subsocial.network/offchain',
    ipfsNodeUrl: 'https://app.subsocial.network/ipfs'
  }
```

  </TabItem>
  <TabItem value="testnet">

```ts
  const config = {
    substrateNodeUrl: 'wss://testnet.subsocial.network',
    offchainUrl: 'https://staging.subsocial.network/offchain',
    ipfsNodeUrl: 'https://staging.subsocial.network/ipfs'
  }
```

  </TabItem>
  <TabItem value="localnet">

```ts
  const config = {
    substrateNodeUrl: 'http://127.0.0.1:9944',
    offchainUrl: 'http://127.0.0.1:3001',
    ipfsNodeUrl: 'http://127.0.0.1:8080'
  }
```
:::caution

Make sure to run local Subsocial & IPFS node before using these configs in your project.

:::
  </TabItem>
</Tabs>

```typescript
  const flatApi = await newFlatSubsocialApi(config)
```

## Reading Data

### Space

Space is the place where all content on SubSocial lives. It holds multiple posts from different people depending upon the permission. [Read More](/docs/glossary/overview#spaces)

<Tabs
  defaultValue="byid"
  values={[
    {label: 'By Id', value: 'byid'},
    {label: 'By Owner', value: 'owner'},
    {label: 'By Handle', value: 'handle'},
  ]}>
  <TabItem value="byid">

```ts
  const spaceId = 1
  const space = await flatApi.findSpace({id: spaceId})
```

  </TabItem>
  <TabItem value="owner">

```ts
  const ownerAccountId = '<owner_account_public_key>'

  // Fetching ids of all the spaces by owner.
  const spaceIds = await flatApi.subsocial.substrate.spaceIdsByOwner(ownerAccountId)

  // Fetching space data from all ids.
  const spaces = await flatApi.subsocial.findSpaces({ids: spaceIds})
```

  </TabItem>
  <TabItem value="handle">

```ts
  const handleName = 'subsocial'

  // Fetching spaceId by Handle.
  const spaceId = await flatApi.subsocial.substrate.getSpaceIdByHandle(handleName)

  // Fetching space by spaceId.
  const space = await flatApi.findSpace({id: spaceId})
```

  </TabItem>
</Tabs>

Check full docs [here](/docs/sdk/quick-start/spaces/getting-spaces).


### Post

Post is the piece of content that provides value for the readers. It can be some written text, an image, or a video. [Read More](/docs/glossary/overview#posts)

<Tabs
  defaultValue="byid"
  values={[
    {label: 'By Id', value: 'byid'},
    {label: 'By Space Id', value: 'byspaceid'},
  ]}>
  <TabItem value="byid">

```ts
  const postId = 1
  const post = await flatApi.findPost(postId)
```

  </TabItem>
  <TabItem value="byspaceid">

```ts
  const spaceId = 1
  const postIds = await flatApi.subsocial.substrate.postIdsBySpaceId(spaceId)

  const posts = await flatApi.subsocial.findPosts({ids: postIds})
```

  </TabItem>
</Tabs>

Check full docs [here](/docs/sdk/quick-start/posts/getting-posts).

### Profile

Profile is linked to your Subsocial account address, and is an overview of your activity on Subsocial. You can set a profile picture and a username for your account, as well as a personal website link. 
[Read More](/docs/glossary/overview#profile)

<Tabs
  defaultValue="singleprofile"
  values={[
    {label: 'Single Account', value: 'singleprofile'},
    {label: 'Multiple Accounts', value: 'multipleprofiles'},
  ]}>
  <TabItem value="singleprofile">

```ts
  const accountId = '<account_public_key>'
  const profile = await flatApi.findProfile(accountId)
```

  </TabItem>
  <TabItem value="multipleprofiles">

```ts
  const accountIds = ['<account_public_key_1>', '<account_public_key_2>']
  const profiles = await flatApi.findProfiles(accountIds)
```

  </TabItem>
</Tabs>

Check full docs [here](/docs/sdk/quick-start/profiles/getting-profiles).

## Writing Data

### Space

Add following import statement: 
```ts
  import {
    IpfsContent, 
    OptionBool,
    SpaceUpdate
  } from "@subsocial/types/substrate/classes" 
```

Storing space details in IPFS, and generating a CID.

```ts
  const ipfsImageCid = await api.subsocial.ipfs.saveFile(file)

  const cid = await ipfs.saveContent({
    about: 'Subsocial is an open protocol for decentralized social networks and marketplaces. It`s built with Substrate and IPFS',
    image: ipfsImageCid, 
    name: 'Subsocial',
    tags: [ 'subsocial' ]
  })
```

Creating a Space transaction object

<Tabs
  defaultValue="create"
  values={[
    {label: 'Create Space', value: 'create'},
    {label: 'Update Space', value: 'update'},
  ]}>
  <TabItem value="create">

```ts
  const substrateApi = await api.subsocial.substrate.api
  const spaceTransaction = substrateApi.tx.spaces.createSpace(
    null, // Parent Id (optional)
    null, // Handle name (optional)
    IpfsContent(cid),
    null // Permissions config (optional)
  )
```

  </TabItem>
  <TabItem value="update">

```ts
  const substrateApi = await api.subsocial.substrate.api
  const update = new SpaceUpdate({
    content: IpfsContent(cid),
    hidden: OptionBool(true),
  })

  const spaceTransaction = substrateApi.tx.spaces.updateSpace('1', update)
```

  </TabItem>
</Tabs>

Signing a transaction and sending to blockchains

<TransactionSnippet variableName="spaceTransaction" />

Check full docs [here](/docs/sdk/quick-start/spaces/creating-spaces).

### Post

Add following import statement: 
```ts
  import {
    IpfsContent, 
    OptionBool,
    SpaceUpdate
  } from "@subsocial/types/substrate/classes" 
```

Storing post details in IPFS, and generating a CID.

```ts
  const ipfsImageCid = await api.subsocial.ipfs.saveFile(file)

  const cid = await ipfs.saveContent({
    title: "What is Subsocial?",
    image: ipfsImageCid,
    tags: [ 'Hello world', 'FAQ' ],
    body: 'Subsocial is an open protocol for decentralized social networks and marketplaces. It`s built with Substrate and IPFS.'
  })
```

Creating a post transaction object

<Tabs
  defaultValue="regular"
  values={[
    {label: 'Regular Post', value: 'regular'},
    {label: 'Shared Post', value: 'shared'},
    {label: 'Update Post', value: 'update'},
  ]}>
  <TabItem value="regular">

```ts
  const spaceId = '1' // The space in which you're posting.
  const substrateApi = await api.subsocial.substrate.api
  const postTransaction = substrateApi.tx.posts.createPost(
    spaceId, 
    { RegularPost: null }, // Creates a regular post.
    IpfsContent(cid)
  )
```

  </TabItem>
  <TabItem value="shared">

```ts
  const spaceId = '1' // The space in which you're posting.
  const parentPostId = '2' // The original post you want to share.

  // Creating new sharedPostCid having shared message.
  const sharedPostCid = await ipfs.saveContent({
    body: 'Keep up the good work!'
  })

  const substrateApi = await api.subsocial.substrate.api
  const postTransaction = substrateApi.tx.posts.createPost(
    spaceId, 
    { SharedPost: parentPostId }, // Creates a shared post.
    IpfsContent(sharedPostCid)
  )
```

  </TabItem>
  <TabItem value="update">

```ts
  const postId = '7' // Id of post which you want to update.
  const substrateApi = await api.subsocial.substrate.api

  const update = new PostUpdate({
    content: IpfsContent(cid),
    hidden: OptionBool(true),
  })

  const postTransaction = substrateApi.tx.spaces.posts.updatePost(postId, update)
```

  </TabItem>
</Tabs>

Signing a transaction and sending to blockchain

<TransactionSnippet variableName="postTransaction" />

Check full docs [here](/docs/sdk/quick-start/posts/creating-posts).

### Profile

Add the following import statement:

```ts
  import { IpfsContent } from "@subsocial/types/substrate/classes"
```

Storing profile details in IPFS, and generating a CID to add on blockchain:

```ts
  const ipfsImageCid = await api.subsocial.ipfs.saveFile(file)
  const cid = await ipfs.saveContent({
    about: 'Subsocial official account.',
    avatar: ipfsImageCid,
    name: 'Subsocial',
  })
```

Creating profile transaction object:

<Tabs
  defaultValue="createprofile"
  values={[
    {label: 'Create Profile', value: 'createprofile'},
    {label: 'Update Profile', value: 'updateprofile'},
  ]}>
  <TabItem value="createprofile">

```ts
  const substrateApi = await api.subsocial.substrate.api
  const profileTransaction = substrateApi.tx.profiles.createProfile(
    IpfsContent(cid)
  )
```

  </TabItem>
  <TabItem value="updateprofile">

```ts
  const update = { content: IpfsContent(cid) }
  const profileTransaction = substrateApi.tx.profiles.updateProfile(update)
```

  </TabItem>
</Tabs>

Signing a transaction and sending to blockchain

<TransactionSnippet variableName="profileTransaction" />

Check full docs [here](/docs/sdk/quick-start/profiles/creating-profiles).

## Comments

Comments are replies to a post that are visible below a post.

### Reading Comments

```ts
  import { idToBn } from "@subsocial/utils"

  const substrate = flatApi.subsocial.substrate
  const postId = '1'

  // Get reply ids (comments) by parent post id and fetch posts by ids
  const replyIds = await substrate.getReplyIdsByPostId(idToBn(postId))

  // For getting comments use posts functions
  const replies = await flatApi.findPublicPosts(replyIds)
```

### Writing Comments

<Tabs
  defaultValue="commentToPost"
  values={[
    {label: 'Create Profile', value: 'commentToPost'},
    {label: 'Update Profile', value: 'replyToComment'},
  ]}>
  <TabItem value="commentToPost">

```ts
  import { IpfsContent } from "@subsocial/types/substrate/classes"

  const spaceId = '1' // Optional.
  const rootPostId = '1'
  const cid = await ipfs.saveContent({
    body: 'Keep up the good work!'
  })

  const substrateApi = flatApi.subsocial.substrate

  const tx = await substrateApi.tx.posts.createPost(spaceId, { Comment: { parentId: null, rootPostId}}, IpfsContent(cid))
```

  </TabItem>
  <TabItem value="replyToComment">

```ts
  import { IpfsContent } from "@subsocial/types/substrate/classes"

  const spaceId = '1' // Optional.
  const parentId = '2' // Parent comment id.
  const rootPostId = '1'
  const cid = await ipfs.saveContent({
    body: 'Agree' // Reply message.
  })

  const substrateApi = flatApi.subsocial.substrate

  const tx = substrateApi.tx.posts.createPost(spaceId, { Comment: { parentId, rootPostId}}, IpfsContent(cid))
```

  </TabItem>
</Tabs>

Check full docs [here](/docs/sdk/quick-start/comments/getting-comments).

## Follows

### Check if follower

This checks if an account is following a particular space.

<Tabs
  defaultValue="isSpaceFollower"
  values={[
    {label: 'Is Space Follower', value: 'isSpaceFollower'},
    {label: 'Is Account Follower', value: 'isAccountFollower'},
  ]}>
  <TabItem value="isSpaceFollower">

```ts
  const accountId = '<any_public_key>'
  const spaceId = '1'

  const substrateApi = flatApi.subsocial.substrate
  const isFollower = await substrateApi.isSpaceFollower(accountId, spaceId)
```

  </TabItem>
  <TabItem value="isAccountFollower">

```ts
  const yourAccountId = '<any_public_key>'
  const otherAccountId = '<any_public_key>'

  const substrateApi = flatApi.subsocial.substrate
  const isFollower = await substrateApi.isAccountFollower(yourAccountId, otherAccountId)
```

  </TabItem>
</Tabs>

### Fetch list of followers

#### For Spaces

<Tabs
  defaultValue="spacefollowers"
  values={[
    {label: 'By Space Id', value: 'spacefollowers'},
    {label: 'Followed by Account Id', value: 'replyToComment'},
  ]}>
  <TabItem value="spacefollowers">

```ts
  import { bnsToIds } from '@subsocial/utils'

  const spaceId = '1'
  const substrateApi = flatApi.subsocial.substrate
  const res = await (await substrateApi.api).query.spaceFollows.spaceFollowers(spaceId)
  const followersSpaceIds = bnsToIds(res)
```

  </TabItem>
  <TabItem value="replyToComment">

```ts
  import { bnsToIds } from '@subsocial/utils'

  const accountId = '<any_public_key>'
  const substrateApi = flatApi.subsocial.substrate
  const res = await (await substrateApi.api).query.spaceFollows.spacesFollowedByAccount(accountId)
  const followedSpaceIds = bnsToIds(res)
```

  </TabItem>
</Tabs>


#### For Accounts

<Tabs
  defaultValue="spacefollowers"
  values={[
    {label: 'Followers', value: 'spacefollowers'},
    {label: 'Following', value: 'replyToComment'},
  ]}>
  <TabItem value="spacefollowers">

```ts
  import { bnsToIds } from '@subsocial/utils'

  const accountId = '<any_public_key>'
  const substrateApi = flatApi.subsocial.substrate
  const res = await (await substrateApi.api).query.profileFollows.accountFollowers(accountId)
  const followersOfAccount = bnsToIds(res)
```

  </TabItem>
  <TabItem value="replyToComment">

```ts
  import { bnsToIds } from '@subsocial/utils'

  const accountId = '<any_public_key>'
  const substrateApi = flatApi.subsocial.substrate
  const res = await (await substrateApi.api).query.profileFollows.accountsFollowedByAccount(accountId)
  const followingOfAccount = bnsToIds(res)
```

  </TabItem>
</Tabs>

Check full docs [here](/docs/sdk/quick-start/follow/getting-follow).

### Follow / Unfollow

#### For Spaces

<Tabs
  defaultValue="spacefollowers"
  values={[
    {label: 'Follow', value: 'spacefollowers'},
    {label: 'Unfollow', value: 'replyToComment'},
  ]}>
  <TabItem value="spacefollowers">

```ts
  const spaceId = '1'
  const substrateApi = flatApi.subsocial.substrate
  const tx = substrateApi.tx.spaceFollows.followSpace(spaceId)
```

  </TabItem>
  <TabItem value="replyToComment">

```ts
  const spaceId = '1'
  const substrateApi = flatApi.subsocial.substrate
  const tx = substrateApi.tx.spaceFollows.unfollowSpace(spaceId)
```

  </TabItem>
</Tabs>

Signing a transaction and sending to blockchain

<TransactionSnippet variableName="tx" />

#### For Accounts

<Tabs
  defaultValue="spacefollowers"
  values={[
    {label: 'Follow', value: 'spacefollowers'},
    {label: 'Unfollow', value: 'replyToComment'},
  ]}>
  <TabItem value="spacefollowers">

```ts
  const accountIdToFollow = '<any_public_key>'
  const substrateApi = flatApi.subsocial.substrate
  const tx = substrateApi.tx.profileFollows.followAccount(accountIdToFollow)
```

  </TabItem>
  <TabItem value="replyToComment">

```ts
  const accountIdToFollow = '<any_public_key>'
  const substrateApi = flatApi.subsocial.substrate
  const tx = substrateApi.tx.profileFollows.followAccount(accountIdToFollow)
```

  </TabItem>
</Tabs>

Signing a transaction and sending to blockchain

<TransactionSnippet variableName="tx" />

Check full docs [here](/docs/sdk/quick-start/follow/following).

## Reactions

Reactions are your signs to `Upvote` or `Downvote` a post.

### Get all reactions

<Tabs
  defaultValue="single"
  values={[
    {label: 'Single Reaction', value: 'single'},
    {label: 'Multiple Reactions', value: 'multi'},
  ]}>

  <TabItem value="single">

```ts
  const myAccount = '<any_account_public_key>';
  const reaction = await flatApi.substrate.getPostReactionIdByAccount (myAccount, '1')
```

  </TabItem>
  <TabItem value="multi">

```ts
  import { ReactionId } from '@subsocial/types/substrate/interfaces';
  
  const myAccount = '<any_account_public_key>';

  const substrate = await flatApi.subsocial.substrate
  const substrateApi = await flatApi.subsocial.substrate.api
  
  const tuples = [ '1', '2', '3' ].map(postId => [ myAccount, postId ])
  
  const reactionIds = await substrateApi.query.reactions.postReactionIdByAccount.multi(tuples)
  const reactions = await substrate.findReactions(reactionIds as ReactionId[])
```

  </TabItem>
</Tabs>

### Reacting to a post

<Tabs
  defaultValue="createReaction"
  values={[
    {label: 'Create', value: 'createReaction'},
    {label: 'Update', value: 'updateReaction'},
    {label: 'Delete', value: 'deleteReaction'},
  ]}>
  <TabItem value="createReaction">

```ts
  const postId = '1' // Post Id you want to react on.
  const substrateApi = flatApi.subsocial.substrate

  const reactionTx = substrateApi.tx.reactions.createPostReaction(postId, 'Upvote')
```

  </TabItem>
  <TabItem value="updateReaction">

```ts
  const postId = '1' // Post Id you want to update reaction on.
  const reactionId = '2' // Reaction Id to update.
  const substrateApi = flatApi.subsocial.substrate

  const reactionTx = substrateApi.tx.reactions.updatePostReaction(postId, reactionId, 'Downvote')
```

  </TabItem>
  <TabItem value="deleteReaction">

```ts
  const postId = '1' // Post Id on which reaction you want to delete reaction.
  const reactionId = '2' // Reaction Id to delete.
  const substrateApi = flatApi.subsocial.substrate

  const reactionTx = substrateApi.tx.reactions.deletePostReaction(postId, reactionId)
```

  </TabItem>
</Tabs>

Signing a transaction and sending to blockchain

<TransactionSnippet variableName="tx" />

Check full docs [here](/docs/sdk/quick-start/reactions/creating-reactions).
