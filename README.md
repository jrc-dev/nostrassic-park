# Nostrassic Park Sub Protocol
Unlocking Decentralized Ratings, Reviews and Movies list with a sub protocol powered by the [**Nostr Protocol**](https://github.com/nostr-protocol/nostr). 🍿🦖

Operates on the Nostr Protocol ensuring that no centralized company controls the content, reviews and ratings.

## Key Features

- **Decentralization:** Operates on the Nostr Protocol, ensuring that no central authority controls the content or reviews. Decentralization fosters real diversity and independence.

- **User-Powered Ratings:** Your ratings and preferences matter. Nostrassic Park relies on the collective wisdom of users you follow, allowing you to discover content that resonates with your unique taste.

- **Enhanced Trust:** By basing recommendations on the ratings of people in your network, Nostrassic Park aims to create a more trustworthy platform where you can confidently explore movies and series.

## Why ?

In the world of entertainment, reviews can be a battleground for differing opinions and cultural narratives. It's not uncommon to encounter instances where movies and series become embroiled in what's often referred to as the "cultural war," where diverse perspectives clash.

**Corporate interests** often play a significant role in shaping the perception of movies and series. Some companies may attempt to influence reviews to promote their content or downplay negative feedback. Understanding these dynamics can help users make informed choices.

Ex: [Rotten Tomatoes Under Fire After PR Firm's Scheme to Pay Critics for Positive Reviews Uncovered](https://www.ign.com/articles/rotten-tomatoes-under-fire-after-pr-firms-scheme-to-pay-critics-for-positive-reviews-uncovered)

Ex: [Is ‘The Rings of Power’ Getting Review Bombed? Amazon Suspends Ratings](https://www.hollywoodreporter.com/tv/tv-news/lord-of-the-rings-the-rings-of-power-amazon-review-bombed-1235211190/)

**Critics and reviewers** also carry significant weight in shaping public opinion. While many critics offer thoughtful, unbiased analysis, some may have **ideological or biased motivations** that influence their reviews. It's essential to approach reviews with a critical eye and consider multiple viewpoints.

## Very short summary of how it works, if you don't plan to read anything else:

Rating a movie is an event in Nostr Protocol with metadatas about the rate and the movie using Proof of Work([NIP-13](https://github.com/nostr-protocol/nips/blob/master/13.md)).

To create a rating for a movie, we create an event of type rating and that you are the author.

To get a movie's rating, we look for a rating-type event and that you are the author.

To remove a rating from a movie, we create an event deletion [NIP-09](https://github.com/nostr-protocol/nips/blob/master/09.md) of kind 5 for the note event.

To calculate a movie's rating, we look for all the ratings of those you follow([NIP-02](https://github.com/nostr-protocol/nips/blob/master/02.md)) and calculate the average.

- **Create a Rating Event:**
```javascript
let event = {
    kind: 1985,
    pubkey: pubkey,
    tags: [
        ['t', movie.id],
        ['l', 'nostr-movie/rating', movie.id, `{"quality": 0.5}`],
        ['l', 'ImdbId', movie.imdbId],
        ['l', 'tmdbId', movie.tmdbId],
        ['l', 'name', movie.name],
        ['l', 'year', movie.year],
        ['l', 'postReviewId', movie?.postReviewId],
    ],
    content: `Just publishing the metadata rating for ${movie.name}.`
  };
  event = await Wallet.mineEvent(event, 15, 5000); //PoW NIP-13
  event = await Wallet.signEvent(event);
....
```

- **Finding my Rating Event:**
```javascript
const event = await relay.get(
    {
        kinds: [1985],
        authors: [pubkey],
        '#t': [movieIdentifier]
    }
);
```

- **Remove rating Event:**
```javascript
let eventDelete = {
    kind: 5, //NIP-09
    pubkey: pubkey,
    tags: [
        ['e', event.id],
        ['t', movieId]
    ],
    content: `Just removing movie rating event.`,
    created_at: Math.round(Date.now() / 1000)
};
```

- **To calculate a movie's rating:**
```javascript
const myContactList = await relay.get(
    {
        kinds: [3],
        authors: [pubkey]
    }
);
const myFriendsReviewPromises = myContactList.map((pub) => {
    return findRatingById({ pubkey, movieId});
});
const myFriendsRating = await Promise.all(myFriendsReviewPromises);
const sumOfRates = myFriendsRating.reduce((accumulator, rate) => {
    const rate = getRateQuality(rate);
    return accumulator + rate;
}, 0);
.....
async function findRatingById(e) {
  const event = await relay.get(
      {
          kinds: [1985],
          authors: [pubkey],
          '#t': [movieId]
      }
  );
  return event;
}

```

