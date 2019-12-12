# Go_Issues
How to retrieve a variable value that includes a "/" from mux package in GO program

I have a go program as below.

package main

import (
	"database/sql"
	//"encoding/json"
	"fmt"
	"github.com/gorilla/mux"
	_ "github.com/lib/pq"
	"log"
	"net/http"
)

const (
	host      = "localhost"
	port      = 5432
	user      = "postgres"
	password1 = "123"
	dbname    = "auzziodb"
)

func SuggestedAddress(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)

	searchstring := vars["searchstring"]
	for k, v := range mux.Vars(r) {
		log.Printf("key=%v, value=%v", k, v)
	}
	fmt.Println(searchstring)
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s "+
		"password=%s dbname=%s sslmode=disable",
		host, port, user, password1, dbname)
	db, _ := sql.Open("postgres", psqlInfo)
	defer db.Close()
	sqlStatement := `select jsonbdata from public."Address_Search" where document @@ to_tsquery($1);`
	rows, err := db.Query(sqlStatement, searchstring)
	if err != nil {
		panic(err)
	}
	defer rows.Close()
	for rows.Next() {
		var full_address string
		err = rows.Scan(&full_address)
		if err != nil {
			panic(err)
		}
		//userJson, _ := json.Marshal(full_address)

		//fmt.Fprintf(w,userJson)
		w.Header().Set("Content-Type", "appliation/json")
		w.WriteHeader(http.StatusOK)
		fmt.Fprintf(w, full_address+"\n")
		w.WriteHeader(http.StatusOK)
		//w.Write(userJson)

	}

}

func main() {
	router := mux.NewRouter().StrictSlash(false)
	//handlerequests()
	router.HandleFunc("/address/search/{searchstring}", SuggestedAddress).Methods("GET")
	router.HandleFunc("/address/search/{searchstring}/", SuggestedAddress).Methods("GET")
	log.Fatal(http.ListenAndServe(":8081", router))
}

I have a database table that stores all the search strings for address using a tsvector column. The table has one address which include a string "4/11". Based on the above code , the url to search the address is http://localhost:8081/address/search/{searchstring} and 
searchstring = "4/11";

The sql @line39 works fine when I pass the search string "4/11" (full text search) , but my URL is not working. I understand that this is becuase we have a "/" in the search string. Is there a way I can pass "/" to the mux variable ?

Thanks
CoderBoy
