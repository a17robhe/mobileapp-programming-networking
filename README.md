
# Rapport

## List view
A new UI element was added to the UI of the main activity. This was done
by adding the list view to the ```activity_main.xml``` file. An ID was
assigned to the list view as well, as we will need it later.
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ListView
        android:id="@+id/list"
        android:layout_width="match_parent"

        android:layout_height="match_parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
## ArrayList and ArrayAdapter
To display a list of items in the list view we need a data structure to
contain these elements, for this a ArrayList object was created in the
java file associated with the main activity. Then to aggregate the
contents to the list view we need an array adapter. This adapter works
as an interface between the list view and the array list, and is used
for, among other things, notifying the list view that the contents of
the array list has changed. The adapter also defines which UI class
should be used to represent the mountain in the array list.
```java
        mountains = new ArrayList<Mountain>();
        adapter = new ArrayAdapter<Mountain>(this, android.R.layout.simple_list_item_1, mountains);

        ListView listView = (ListView) findViewById(R.id.list);
        listView.setAdapter(adapter);
```
## The mountain class
To properly represent the data fetched from the json webservice
programmatically we need a model class to contain the data. This was
done by creating a basic container class with getters and a constructor,
this is the only functionality that the model class would need. The
```toString()``` function was also overridden to display the mountains
in the list correctly, it will simply return the name of the mountain as
a string.
```java
package com.example.networking;

public class Mountain {
    private String ID;
    private String name;
    private String type;
    private String company;
    private String location;
    private String category;
    private int size;
    private int cost;
    private String wiki;
    private String img;

    public Mountain(String id, String name, String type, String company, String location, String category, int size, int cost, String wiki, String img) {
        ID = id;
        this.name = name;
        this.type = type;
        this.company = company;
        this.location = location;
        this.category = category;
        this.size = size;
        this.cost = cost;
        this.wiki = wiki;
        this.img = img;
    }

    public String getID() {
        return ID;
    }

    public String getName() {
        return name;
    }

    public String getType() {
        return type;
    }

    public String getCompany() {
        return company;
    }

    public String getLocation() {
        return location;
    }

    public String getCategory() {
        return category;
    }

    public int getSize() {
        return size;
    }

    public int getCost() {
        return cost;
    }

    public String getWiki() {
        return wiki;
    }

    public String getImg() {
        return img;
    }

    @Override
    public String toString() {
        return name;
    }
}

```
## JsonTask
The assignment provides a class with a stub method for fetching the json
from the web service. The class simply provides the functionality for
fetching the json data as a string object, we will need to implement the
unmarshal portion that creates an instance of the model class and adds
it to the array list.
```java
  @Override
        protected void onPostExecute(String json) {
            try {
                JSONArray arr = (JSONArray) new JSONTokener(json).nextValue();
                for (int i = 0; i < arr.length(); i++){
                    JSONObject element = arr.optJSONObject(i);
                    String ID = element.getString("ID");
                    String name = element.getString("name");
                    String type = element.getString("type");
                    String company = element.getString("company");
                    String location = element.getString("location");
                    String category = element.getString("category");
                    int size = element.getInt("size");
                    int cost = element.getInt("cost");
                    String wiki = element.getJSONObject("auxdata").getString("wiki");
                    String img = element.getJSONObject("auxdata").getString("img");

                    mountains.add(new Mountain(ID,name,type,company,location,category,size,cost,wiki,img));
                }
                adapter.notifyDataSetChanged();
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
```
Here we use the JSON - Simple library to create JSON object and JSON
array objects from the string. The library was added by adding the
dependency to the build.gradle file. The code uses a JSONTokener to
convert the string to a JSON array, it is then iterated over and each
JSON object in the array has its data extracted. The data is then used
to create a new mountain class which is added to the array list in the
main activity. When all the objects have been unmarshalled, the adapter
is notified of the change so the list view gets updated. Finally a
JsonTask object is created and its execute function is called in the
main activity's ```onCreate``` function, passing the url to the
webservice to the function.

## List interactivity

Then interactivity is added to the list items displayed in the list
view. This is simply done by adding a ```onClick``` function to the list
item. In this case the function simply creates a snackbar notification
that is displayed for a long duration but has its own ```onClick```
function defined that dismisses the snackbar. The snackbar simply
displays the name, location and size of the mountain clicked, the
listener gets passed the mountains position in the array list which is
used to fetch the relevant data.
```java
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener(){
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                Mountain clickedMountain = mountains.get(position);
                
                final Snackbar snackbar = Snackbar.make(findViewById(R.id.list),
                        clickedMountain.getName() + " - " + clickedMountain.getLocation() + ": " + clickedMountain.getSize(),
                        999999 );
                snackbar.setAction("Dismiss", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        snackbar.dismiss();
                    }
                });
                snackbar.show();
            }
        });
```
