Approach:

Instead of storing all the salt level data in a single redis hashmap, we plan to keep two hash maps.

First hashmap would just store resIds corresponding to each offer type with redis key as <S2Cell_OfferType_Date> and hashKey as resId and value as true. Since the data is very less, the size won’t exceed 2-3 mb. 

In the second hashmap, we’ll keep resId level offer details. Redis key: ResId_Date and hashMap key as offerType_saltId.

At any given point of time, we’ll maintain the hash maps for current date and the next day

Similar to res indexing, every night a cron would publish all the res ids in a Kafka topic which will be consumed in search service.

Upon consuming the res event, we’ll make a call to benefits service corresponding to specific res and fetch all the offers running on it.

After fetching the offer details, we’ll populate the resId in all the relevant <S2Cell_OfferType_Date> hashmaps.

Correspondingly, we’ll store the res metadata like timeslots, tagged dish ids in another hash map.

For eg. - resId is 20208 and today’s date is 08/01/25
Offer details for resId 20208 is as follows:

{
    resId: 20208
    saltId: 10102
    timeslots : [[12:00,14:00],[16:00,18:00]]
    item_ids: [1,2,3,4]
    offertype: BXGY
}
