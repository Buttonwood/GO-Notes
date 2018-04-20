####  Application design

![img1](https://dpzbhybb2pdcj.cloudfront.net/chang/Figures/02fig04_alt.jpg)

```
http://<servername>/<handlername>?<parameters>
```

#### Data model

![img2](https://dpzbhybb2pdcj.cloudfront.net/chang/Figures/02fig05_alt.jpg)

```
User 	//Representing the forum user’s information
Session //Representing a user’s current login session
Thread  //Representing a forum thread (a conversation among forum users)
Post 	//Representing a post (a message added by a forum user) within a thread
```

```
package main

import (
  "net/http"
)

func main() {
	mux := http.NewServeMux()
	
	// Serving static files
	files := http.FileServer(http.Dir("/public"))
	mux.Handle("/static/", http.StripPrefix("/static/", files))
	
	// To redirect the root URL to a handler function
	mux.HandleFunc("/", index)

	server := &http.Server{
		Addr:     "0.0.0.0:8080",
		Handler:  mux,
  	}
  	server.ListenAndServe()
}

func index(w http.ResponseWriter, r *http.Request) {
	files := []string{"templates/layout.html",
		"templates/navbar.html",
		"templates/index.html",
	}
	templates := template.Must(template.ParseFiles(files...))
	threads, err := data.Threads(); if err == nil {
		templates.ExecuteTemplate(w, "layout", threads)
	}
	
}
```

*	You use the `StripPrefix` function to remove the given prefix from the request URL’s path.

*	 You don’t need to provide the parameters to the handler function because all handler functions take `ResponseWriter` as the first parameter and a pointer to `Request` as the second parameter.