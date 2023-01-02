---
layout: post
title: "Handling Dynamic JSONAPI Format With Retrofit in Android"
date: 2015-10-16 13:31:35 +0700
comments: true
categories: Tips
---

Sometimes working with json can be frustrating if we don’t understand how the dependencies work such as retrofit and JSONAPI.org

<!--more-->

Look on this json format

``` json
{
  "data": [
    {
      "id": "561c7ef2eab7094edd000019",
      "type": "orders",
      "attributes": {
        "inquiry_ticket": 65074547,
        "invoice_number": 120033,
        "unique_code": 702,
        "custom_vendor": null,
        "cod": false,
        "payment_status": "unpaid",
        "payment_time": null,
        "status": "initial",
        "created_at": "2015-10-13T10:48:02.494+07:00",
        "updated_at": "2015-10-13T10:48:02.494+07:00"
      },
      "links": {
        "self": "/orders/561c7ef2eab7094edd000019"
      },
      "relationships": {
        "user": {
          "links": {
            "self": "/orders/561c7ef2eab7094edd000019/relationships/user",
            "related": "/orders/561c7ef2eab7094edd000019/user"
          }
        },
        "inquiry": {
          "links": {
            "self": "/orders/561c7ef2eab7094edd000019/relationships/inquiry",
            "related": "/orders/561c7ef2eab7094edd000019/inquiry"
          }
        },
        "payment_confirm": {
          "links": {
            "self": "/orders/561c7ef2eab7094edd000019/relationships/payment_confirm",
            "related": "/orders/561c7ef2eab7094edd000019/payment_confirm"
          }
        },
        "sub_category": {
          "links": {
            "self": "/orders/561c7ef2eab7094edd000019/relationships/sub_category",
            "related": "/orders/561c7ef2eab7094edd000019/sub_category"
          }
        },
        "category": {
          "links": {
            "self": "/orders/561c7ef2eab7094edd000019/relationships/category",
            "related": "/orders/561c7ef2eab7094edd000019/category"
          },
          "data": {
            "type": "categories",
            "id": "561491daeab7090bb4000001"
          }
        },
        "ordered_items": {
          "links": {
            "self": "/orders/561c7ef2eab7094edd000019/relationships/ordered_items",
            "related": "/orders/561c7ef2eab7094edd000019/ordered_items"
          },
          "data": [
            {
              "type": "ordered-items",
              "id": "560a523aeab709322b000014"
            }
          ]
        }
      }
    }
  ],
  "included": [
    {
      "id": "560a523aeab709322b000014",
      "type": "ordered-items",
      "attributes": {
        "description": "martabk",
        "purchased": 50000,
        "sell": 55000,
        "margin": null
      },
      "links": {
        "self": "/ordered-items/560a523aeab709322b000014"
      }
    },
    {
      "id": "561491daeab7090bb4000001",
      "type": "categories",
      "attributes": {
        "name": "Transportation",
        "cname": "transport",
        "sub_categories": [
          {
            "_id": {
              "$oid": "56161982eab7092dda000001"
            },
            "category_id": {
              "$oid": "561491daeab7090bb4000001"
            },
            "cname": "Transport",
            "created_at": "2015-10-08T14:21:38.236+07:00",
            "name": "Transport",
            "updated_at": "2015-10-08T14:21:38.236+07:00"
          }
        ]
      },
      "links": {
        "self": "/categories/561491daeab7090bb4000001"
      }
    }
  ]
}
```

JSON format is one of recommendation from JSONAPI.org and our company has used this format for the restful api, and now as an Android Developer, I need to parse those format.

On android side, I used retrofit for beautiful type safe Restful for android. I could parse almost everything except 1 object, the ‘included’ object. Now look on that object, and you will realize that it’s an object with array inside it. The problem was that the array is not a single data type. There are multiple type of data inside the array, and retrofit does not really support json array with multiple data types like this.

I believe there is a better way to solve this problem or maybe retrofit v2.0 supports this kind of issue, need more research about this.

As you might guess, I need to use more advanced solution and deep modification on the retrofit and gson side

###Solution

First, I need to have a model that consists of all the data types inside of the json array. In the json array, we have 2 types, the categories and ordered-items, so I created the model like this

``` java
public class IncludedModel extends BaseModel {

    private OrderedItem orderedItem;
    private Category category;

    public Object getObject() {
        switch (getType()) {
            case ModelType.ORDERED_ITEMS:
                return getOrderedItem();
            case ModelType.CATEGORIES:
                return getCategory();
        }

        return null;
    }

    public OrderedItem getOrderedItem() {
        return orderedItem;
    }

    public void setOrderedItem(OrderedItem orderedItem) {
        this.orderedItem = orderedItem;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    @Override
    public String toString() {
        return "IncludedModel{" +
                "orderedItem=" + orderedItem +
                ", category=" + category +
                '}';
    }
}
```

Then I need to create a custom deserializer, and we need this deserializer only for that specific object, and fortunately, Retrofit is really extensible and I can make such modification easily.

So, I created a simple deserializer like this one

``` java
public class IncludedModelDeserializer implements JsonDeserializer<ArrayList<IncludedModel>> {
    @Override
    public ArrayList<IncludedModel> deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        ArrayList<IncludedModel> list = new ArrayList<>();

        JsonArray jsonArray = json.getAsJsonArray();

        for(JsonElement object: jsonArray) {

            LogUtils.d(object.toString());
            JsonObject jsonObject = object.getAsJsonObject();
            String id = jsonObject.get("id").getAsString();
            String type = jsonObject.get("type").getAsString();

            IncludedModel model = new IncludedModel();
            model.setId(id);
            model.setType(type);

            Gson gson = new Gson();
            switch(type) {
                case ModelType.ORDERED_ITEMS:
                    OrderedItem orderedItem = gson.fromJson(jsonObject, OrderedItem.class);
                    model.setOrderedItem(orderedItem);
                    break;
                case ModelType.CATEGORIES:
                    Category category = gson.fromJson(jsonObject, Category.class);
                    model.setCategory(category);
                    break;
            }

            list.add(model);
        }

        return list;
    }
}
```

Look, I implemented JSONDeserializer with `Arraylist<IncludedModel>` as the type. If you look closely, I’m separating JSON Array into each object, and then I parsed manually the object with Gson, and then put it into the model.

After that I created a custom gson with this deserializer, and register the gson to retrofit

``` java
Type type = new TypeToken<ArrayList<IncludedModel>>(){}.getType();
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapter(type, new IncludedModelDeserializer());
Gson gson = gsonBuilder.create();

RestAdapter.Builder builder = new RestAdapter.Builder()
                .setEndpoint(Config.SERVER_NAME)
                .setConverter(new GsonConverter(gson))
                .setLogLevel(RestAdapter.LogLevel.NONE);
```

Then, we just do process as usual, and retrofit will do the rest. If you also use JSONAPI format then I also recommend you to put the included data into Hashmap such as this

``` java
private HashMap<String, IncludedModel> mIncludedList;

private void addIncludedOrderedItemToList(ArrayList<IncludedModel> included) {
  for(IncludedModel item: included) {
    mIncludedList.put(item.getId(), item);
  }
}
```

Because it’s going to be easier for you to get the data based on the id of the object,

``` java
Category category = (Category) mIncludedList.get("id").getObject();
```

Hope it’s useful
