---
layout: post
title: "Handling Dynamic JSONAPI Format With Retrofit in Android"
date: 2015-10-16 13:31:35 +0700
comments: true
categories: Tips
---

Sometimes working with json can be frustrating if we don’t understand how the dependencies work such as retrofit and JSONAPI.org

<!--more-->

Here is sample of one of json response

```json
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

JSON format is one of recommendation from JSONAPI.org. Here Retrofit can parse almost everything out of the box except one array object which is `included` object. There are multiple type of data inside this arrray object, and retorift is not able to parse this data easily.

As you can guess, We need to find another way to parse this kind of json format. For this example I use retrofit and GSON.

### Solution

First, I need to have a model that consists of all the data types inside of the json array. In the json array, we have 2 types, the categories and ordered-items, so I created the model like this

```java
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

Then I create a custom deserializer, and we need this deserializer only for that specific object, and fortunately, Retrofit has feature for this.

Here is how the deserializer looks like

```java
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

First I implement JSONDeserializer with `Arraylist<IncludedModel>` as the data type, and then I’m separating JSON Array into each object, and then I parse manually the object with Gson, and then put it into the model.

After that I just need to register this type adapter to `GsonBuilder`

```java
Type type = new TypeToken<ArrayList<IncludedModel>>(){}.getType();
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.registerTypeAdapter(type, new IncludedModelDeserializer());
Gson gson = gsonBuilder.create();

RestAdapter.Builder builder = new RestAdapter.Builder()
                .setEndpoint(Config.SERVER_NAME)
                .setConverter(new GsonConverter(gson))
                .setLogLevel(RestAdapter.LogLevel.NONE);
```

If you also use JSONAPI format then I also recommend you to put the included data into Hashmap like this

```java
private HashMap<String, IncludedModel> mIncludedList;

private void addIncludedOrderedItemToList(ArrayList<IncludedModel> included) {
  for(IncludedModel item: included) {
    mIncludedList.put(item.getId(), item);
  }
}
```

Because it’s going to be easier to get the data based on the id of the object.

``` java
Category category = (Category) mIncludedList.get("id").getObject();
```
