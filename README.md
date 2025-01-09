This is cheatsheet/primer to get up to speed quickly with Golang's standard library (1.23.4)

https://pkg.go.dev/std


# bufio
https://pkg.go.dev/bufio

https://www.kelche.co/blog/go/golang-bufio/

Reads or writes data in chunks. If reading/writting a lot of data buffered I/O can be more performant. 

Reader:
```
func main() {
	file, err := os.Open("f.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()
	reader := bufio.NewReader(file)
	buf := make([]byte, 5)
	n, err := reader.Read(buf)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%d bytes read. Buf contents are [%s]", n, buf)
}
```

Writer:
```
func main() {
	file, err := os.Create("f.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	_, err = writer.Write([]byte("my data"))
	if err != nil {
		panic(err)
	}
	err = writer.Flush()
	if err != nil {
		panic(err)
	}
}

```
Scanner:
```
func main() {
	file, err := os.Open("f.txt")
	if err != nil {
		panic(err)
	}
	scanner := bufio.NewScanner(file)
	//print each line
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
	if err := scanner.Err(); err != nil {
		panic(err)
	}
}

```

More:
```
//reader with certain buffer size
reader512 := bufio.NewReader(myfile, 512)
//buffered writer with certain buffer size
writer512 := bufio.NewWriterSize(myfile, 512)
```

# builtin

https://pkg.go.dev/builtin

Recover:
```
func mayPanic() {
	panic("problem!!!")
}

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recovered from error ", r)
		}
	}()
	mayPanic()
}

```

More:
```
//append elements to end of a slice
s := []int{1, 2, 3}
s = append(s, 42)

//clear maps and slices
clear(s)

//close channel
close(myChannel)

//copy one slice to another
copy(destSlice, srcSlice)

//delete element with key from map
delete(myMap, myKey)

//make slices, maps, channels
mySlice := make([]int, 10)
myMap := make(map[string]int)
myChannel := make(chan bool)
```

# bytes
https://pkg.go.dev/bytes

Functions for manipulating byte slices, similar to the strings package.
It also has a very useful buffer implementing Reader and Writer interfaces

Buffer:
```
func main() {
	var b bytes.Buffer
	b.Write([]byte("hello world"))
	res, err := io.ReadAll(&b)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(res))
}

```

Buffer from string:
```
	b := bytes.NewBufferString("hello world")
	b.WriteTo(os.Stdout)
```

Read only buffer (Reader):
```
	b := bytes.NewReader([]byte("hello world"))
	//this won't work
	// b.Write([]byte("not allowed"))
	b.WriteTo(os.Stdout)
```

More:
```
	//Contains
	b := []byte("hello world")
	fmt.Println(bytes.Contains(b, []byte("world")))

	//Count
	b := []byte("hello friend")
	fmt.Println(bytes.Count(b, []byte("e")))

	//Index
	b := []byte("hello friend")
	i := bytes.Index(b, []byte("f"))

	//Replace
	//at most one replacement
	b := []byte("hello world")
	old := []byte("world")
	new := []byte("friend")
	fmt.Printf("%s", bytes.Replace(b, old, new, 1))
```

# cmp
https://pkg.go.dev/cmp

More:
```
	//Or
	fmt.Println(cmp.Or(0, 1))
	//Compare
	fmt.Println(cmp.Compare(1, 2))
```
# compress/gzip
https://pkg.go.dev/compress/gzip
```
	//create new file
	gzipF, err := os.Create("f.txt.gz")
	if err != nil {
		panic(err)
	}
	defer gzipF.Close()

	//write data
	gw := gzip.NewWriter(gzipF)
	gw.Name = "f.txt"
	_, err = gw.Write([]byte("data"))
	if err != nil {
		panic(err)
	}

	err = gw.Close()
	if err != nil {
		panic(err)
	}
```

# context
https://pkg.go.dev/context

Timeout:
```
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	select {
	case <-time.After(2 * time.Second):
		fmt.Println("2 seconds passed")
	//this will be called first
	case <-ctx.Done():
		fmt.Println("context timeout")
	}
}

```

Store and get value:
```
func main() {
	type contextKey string
	k := contextKey("myKey")
	ctx := context.WithValue(context.Background(), k, "myValue")

	v := ctx.Value(k)
	fmt.Println(v)
}
```

Call function after context is cancelled/timed out:
```
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	stopf := context.AfterFunc(ctx, func() {
		fmt.Println("I am called after context is cancelled or timed out")
	})
	defer stopf()
	select {
	case <-ctx.Done():
		fmt.Println("context timeout")
	}
}
```

Child context without cancel:
```
	ctx, cancel1 := context.WithCancel(context.Background())
	ctxNoCancel := context.WithoutCancel(ctx)
	//cancel parent ctx
	cancel1()
	//ctxNoCancel is stil alive
```

# crypto/sha256
https://pkg.go.dev/crypto/sha256

SHA256 hash algorithm:
```
	h := sha256.New()
	h.Write([]byte("secret"))
	fmt.Printf("%x", h.Sum(nil))
```

# database/sql
https://pkg.go.dev/database/sql
https://betterstack.com/community/guides/scaling-go/sql-databases-in-go/
https://www.alexedwards.net/blog/configuring-sqldb


Open db and ping to check connection:
```
	dsn := "mydsn"
	pool, err := sql.Open("driver-name", dsn)
	ctx := context.Background()
	if err := pool.PingContext(ctx); err != nil {
		panic(err)
	}
```

Single row query:
```
	var id string
	var name = "John Doe"
	row := db.QueryRowContext(context.TODO(), `SELECT id FROM customer WHERE name = $1`, name)
	err := row.Scan(&id)
	if err != nil {
		if err == sql.ErrNoRows {
			panic("No results")
		}
		panic(err)
	}
	fmt.Println("id is ", id)
```

Multiple row query:
```
	rows, err := db.QueryContext(context.TODO(), `SELECT name FROM customer`)
	if err != nil {
		panic(err)
	}
	defer rows.Close()

	for rows.Next() {
		var name string
		err = rows.Scan(&name)
		if err != nil {
			log.Fatalf("error occured during row retrieval %s", err)
		}
		fmt.Println("customer name is ", name)
	}

	if err := rows.Err(); err != nil {
		log.Fatalf("error occured during iteration %s", err)
	}
```

Modify data:
```
	name := "John"
	_, err := db.ExecContext(context.TODO(), `INSERT INTO customer(name) VALUES($1);`, name)
	if err != nil {
		panic(err)
	}
```

Transactions:
```
	ctx := context.Background()
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		log.Fatalf("failed to start transaction %s", err)
	}
	defer tx.Rollback()

	name := "John"
	_, err = tx.ExecContext(ctx, `INSERT INTO customer(name) VALUES($1);`, name)
	if err != nil {
		panic(err)
	}

	secondName := "Mike"
	_, err = tx.ExecContext(ctx, `INSERT INTO customer(name) VALUES($1);`, secondName)
	if err != nil {
		panic(err)
	}

	tx.Commit()
```

Connection settings:
```
	db.SetMaxOpenConns(30)
	db.SetMaxIdleConns(5)

	db.SetConnMaxLifetime(time.Minute * 100)
	db.SetConnMaxIdleTime(time.Minute * 100)
```

# embed
https://pkg.go.dev/embed

Embedding files:
```
//go:embed internal/embedtest/testdata/*.txt
var content embed.FS

func main() {
	mux := http.NewServeMux()
	mux.Handle("/", http.FileServer(http.FS(content)))
	err := http.ListenAndServe(":8080", mux)
	if err != nil {
		log.Fatal(err)
	}
}
```