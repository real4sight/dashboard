# Making Golang Friendly with MongoDB

## Setting Up Golang & MongoDB

1.  Start golang normally (create src, make a new folder inside, etc)
2.  use the command:

```Go
go get gopkg.in/mgo.v2
```

to get the mongo driver that speaks to the MongoDB server

3.  Download MongoDB from [here](https://www.mongodb.com/download-center) and choosing community service
4.  Add /path/to/mongodb/bin to PATH so you can execute the commands inside
5.  cd to the ```src/<project>``` and create a /db folder inside then use
```cmd
mongod --dbpath="src/<project>/db"
```
6.  A server should run with the mongoDB shell, now you can communicate with it

## Example Code
*taken from [p4tin](https://gist.github.com/p4tin/9bfb778fa7089a82030c)*
```Go
package main

import (
	"fmt"
	"log"
	"time"

	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

var connection = "localhost"
var database = "ProfileService"
var collection = "Profiles"

// Profile - is the memory representation of one user profile
type Profile struct {
	Name        string `json:"username"`
	Password    string `json:"password"`
	Age         int    `json:"age"`
	LastUpdated time.Time
}

func main() {
	p := Profile{Name: "marco", Password: "thePassy", Age: 4, LastUpdated: time.Now()}

	createFile()
	writeFile("helo", "me")

	isCreated := p.CreateOrUpdateProfile()
	fmt.Println(isCreated)
	fmt.Println(ShowProfile("marco").Password)
	fmt.Println(GetProfiles())
}

// GetProfiles - Returns all the profile in the Profiles Collection
func GetProfiles() []Profile {
	session, err := mgo.Dial(connection)
	if err != nil {
		log.Println("Could not connect to mongo: ", err.Error())
		return nil
	}
	defer session.Close()

	// Optional. Switch the session to a monotonic behavior.
	session.SetMode(mgo.Monotonic, true)

	c := session.DB(database).C(collection)
	var profiles []Profile
	err = c.Find(bson.M{}).All(&profiles)

	return profiles
}

// ShowProfile - Returns the profile in the Profiles Collection with name equal to the id parameter (id == name)
func ShowProfile(id string) Profile {
	session, err := mgo.Dial(connection)
	if err != nil {
		log.Println("Could not connect to mongo: ", err.Error())
		return Profile{}
	}
	defer session.Close()

	// Optional. Switch the session to a monotonic behavior.
	session.SetMode(mgo.Monotonic, true)

	c := session.DB(database).C(collection)
	profile := Profile{}
	err = c.Find(bson.M{"name": id}).One(&profile)

	return profile
}

// DeleteProfile - Deletes the profile in the Profiles Collection with name equal to the id parameter (id == name)
func DeleteProfile(id string) bool {
	session, err := mgo.Dial(connection)
	if err != nil {
		log.Println("Could not connect to mongo: ", err.Error())
		return false
	}
	defer session.Close()

	// Optional. Switch the session to a monotonic behavior.
	session.SetMode(mgo.Monotonic, true)

	c := session.DB(database).C(collection)
	err = c.RemoveId(id)

	return true
}

// CreateOrUpdateProfile - Creates or Updates (Upsert) the profile in the Profiles Collection with id parameter
func (p *Profile) CreateOrUpdateProfile() bool {
	session, err := mgo.Dial(connection)
	if err != nil {
		log.Println("Could not connect to mongo: ", err.Error())
		return false
	}
	defer session.Close()

	// Optional. Switch the session to a monotonic behavior.
	session.SetMode(mgo.Monotonic, true)

	c := session.DB(database).C(collection)
	_, err = c.UpsertId(p.Name, p)
	if err != nil {
		log.Println("Error creating Profile: ", err.Error())
		return false
	}
	return true
}

```

## Using MongoDB as API 
*From [here](https://medium.com/@matryer/production-ready-mongodb-in-go-for-beginners-ef6717a77219)*