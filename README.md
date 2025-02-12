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
# encoding/base64
https://pkg.go.dev/encoding/base64

Encoding/Decoding:

```
	msg := "hello world"
	encoded := base64.StdEncoding.EncodeToString([]byte(msg))
	fmt.Printf("encoded value is %s\n", encoded)
	decoded, err := base64.StdEncoding.DecodeString(encoded)
	if err != nil {
		panic(err)
	}
	fmt.Printf("decoded value %s\n", decoded)
```

# encoding/binary
https://pkg.go.dev/encoding/binary

Write binary encoding:
```
	var buf bytes.Buffer
	var pi float64 = math.Pi
	err := binary.Write(&buf, binary.LittleEndian, pi)
	if err != nil {
		panic(err)
	}
	fmt.Printf("% x", buf.Bytes())
```

# encoding/json
https://pkg.go.dev/encoding/json

Basic marshalling and unmarshalling:
```
type Student struct {
	Name string
}

func main() {
	student := Student{"John"}
	b, err := json.Marshal(student)
	if err != nil {
		panic(err)
	}
	var newStudent Student
	err = json.Unmarshal(b, &newStudent)
	if err != nil {
		panic(err)
	}
	fmt.Println(newStudent)
}
```
Encoder:
```
type Student struct {
	Name string
}

func main() {
	student := Student{"John"}
	json.NewEncoder(os.Stdout).Encode(student)
}
```

Decoder:
```
func main() {
	student := Student{"John"}
	var buf bytes.Buffer
	err := json.NewEncoder(&buf).Encode(student)
	if err != nil {
		panic(err)
	}
	var newStudent Student
	err = json.NewDecoder(&buf).Decode(&newStudent)
	if err != nil {
		panic(err)
	}
	fmt.Println(newStudent)
}
```

Decoder that keeps reading a stream of data until EOF:
```
func main() {

	const jsonStream = `
	{"Name": "Ed", "Text": "Knock knock."}
	{"Name": "Sam", "Text": "Who's there?"}
	{"Name": "Ed", "Text": "Go fmt."}
	{"Name": "Sam", "Text": "Go fmt who?"}
	{"Name": "Ed", "Text": "Go fmt yourself!"}
`
	type Message struct {
		Name, Text string
	}
	dec := json.NewDecoder(strings.NewReader(jsonStream))

	for {
		var m Message
		if err := dec.Decode(&m); err == io.EOF {
			break
		} else if err != nil {
			log.Fatal(err)
		}
		fmt.Println(m)
	}
}
```

Custom marshal, change value:
```
type Student struct {
	Name string
}


func (s Student) MarshalJSON() ([]byte, error) {
	type Alias Student
	alias := Alias(s)
	alias.Name = fmt.Sprintf("CustomName %s", alias.Name)
	return json.Marshal(alias)
}

func main() {
	s := Student{"John"}
	b, err := json.Marshal(s)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
}

```

Custom marshal, return a new struct:
```
func (s Student) MarshalJSON() ([]byte, error) {
	type Alias Student
	custStruct := struct {
		LastModified int64 `json:"last_modified"`
		Student      Alias `json:"student"`
	}{
		LastModified: time.Now().Unix(),
		Student:      Alias(s),
	}

	return json.Marshal(custStruct)
}

func main() {
	s := Student{"John"}
	b, err := json.Marshal(s)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
}

```

Custom unmarshal:
```
func (s *Student) UnmarshalJSON(b []byte) error {
	type Alias Student
	var a Alias
	err := json.Unmarshal(b, &a)
	if err != nil {
		return err
	}
	a.Name = "CompletelyNewName"
	*s = Student(a)
	return nil
}

func main() {
	s := Student{"John"}
	b, err := json.Marshal(s)
	if err != nil {
		panic(err)
	}
	var newStudent Student
	err = json.Unmarshal(b, &newStudent)
	if err != nil {
		panic(err)
	}
	fmt.Println(newStudent)
}

```

Raw message:
```
type Student struct {
	Name     string
	SomeData json.RawMessage
}

func main() {
	var data = `
		{
		"Name":"John",
		"SomeData":42
		}
	`
	b := []byte(data)
	var student Student
	json.Unmarshal(b, &student)

	if student.Name == "John" {
		var someData int64
		err := json.Unmarshal(student.SomeData, &someData)
		if err != nil {
			panic(err)
		}
		fmt.Println(someData)
	}
}
```

Unmarshal into a map[string]any:
```
func main() {
	var data = `
		{
		"Name":"John",
		"SomeData":42
		}
	`
	var m = make(map[string]any)
	json.Unmarshal([]byte(data), &m)
	fmt.Println(m)
}
```

# errors
https://pkg.go.dev/errors
https://adrianlarion.com/golang-error-handling-demystified-errors-is-errors-as-errors-unwrap-custom-errors-and-more/


Is:
```
var ErrMyCustom = errors.New("my custom error")

func main() {
	err := do()
	if errors.Is(err, ErrMyCustom) {
		fmt.Println("we have a custom error")
	}
}

func do() error {
	return ErrMyCustom
}
```

Is for wrapped errors (still works even thogh the error is wrapped):
```
var ErrMyCustom = errors.New("my custom error")

func main() {
	err := do()
	if errors.Is(err, ErrMyCustom) {
		fmt.Println("we have a custom error")
	}
}

func do() error {
	return fmt.Errorf("additional information and the original err %w", ErrMyCustom)
}
```

As:
```
type CustomErr struct {
	ExtraInfo string
	Err       error
}

func (c CustomErr) Error() string {
	return fmt.Sprintf("Extra info is '%s' and original err is %v", c.ExtraInfo, c.Err)
}

func main() {
	err := do()
	var customErr CustomErr
	if errors.As(err, &customErr) {
		fmt.Println(" we have a custom error")
	}
}

func do() error {
	customErr := CustomErr{ExtraInfo: "my extra info", Err: errors.New("my original err")}
	return customErr
}

```

Join:
```
var err1 = errors.New("err1")
var err2 = errors.New("err2")
err := errors.Join(err1, err2)
fmt.Println(err)
if errors.Is(err, err1) {
	fmt.Println("we have err1")
}
if errors.Is(err, err2) {
	fmt.Println("we have err2")
}
```

# expvar
https://pkg.go.dev/expvar
https://sysdig.com/blog/golang-expvar-custom-metrics/

Publish:
```
type Metrics struct {
	Metric1 float64
	Metric2 float64
}

func MyMetrics() any {
	return Metrics{42, 43}
}

func main() {
	expvar.Publish("system.metrics", expvar.Func(MyMetrics))
	http.ListenAndServe(":8080", nil)
}
```

More:
```
func main() {
	var fooCount = expvar.NewInt("foo.count")
	fooCount.Add(1)
	http.ListenAndServe(":8080", nil)
	// access http://localhost:8080/debug/vars for the metrics

}
```

# flag
https://pkg.go.dev/flag

Parse:
```
	var target = flag.String("target", "defaultValue", "medical code")
	flag.Parse()
	fmt.Println(*target)
```

# fmt
https://pkg.go.dev/fmt

Stringer interface:
```
type Student struct {
	Name  string
	Money int64
}

func (s Student) String() string {
	return fmt.Sprintf("student %s has %v money", s.Name, s.Money)
}

func main() {
	s := Student{"john", 42}
	fmt.Println(s)
}
```

More:
```
func main() {

	fmt.Println("hello")

	fmt.Printf("%v is the answer", 42)

	s := fmt.Sprintf("%v is the answer", 42)
	fmt.Println(s)

	err := fmt.Errorf("my error and extra info %v", 42)
	fmt.Println(err)

	fmt.Fprintln(os.Stdout, "hello")

	var a string
	_, err = fmt.Scan(&a)
	if err != nil {
		panic(err)
	}
	fmt.Printf("a is %v", a)
}
```

# io
https://pkg.go.dev/io

Arguably the most important interfaces of Go's stdlib:
```
//write to underlying data stream 'p' bytes
type Writer interface {
	Write(p []byte) (n int, err error)
}
...
//read from underlying data stream into 'p' bytes
type Reader interface {
	Read(p []byte) (n int, err error)
}

```

Copy to dst writer from src reader:
```
	var b bytes.Buffer
	b.Write([]byte("hello\n"))
	//copy to dst writer (os.Stdout) from source reader (&b)
	io.Copy(os.Stdout, &b)
```

Multi reader:
```
	r1 := strings.NewReader("hello ")
	r2 := strings.NewReader("world\n")

	multi := io.MultiReader(r1, r2)
	io.Copy(os.Stdout, multi)
```

Multi writer:
```
	var b1 bytes.Buffer
	var b2 bytes.Buffer
	w := io.MultiWriter(&b1, &b2)
	w.Write([]byte("hello world\n"))
	fmt.Printf("%s", b1.Bytes())
	fmt.Printf("%s", b2.Bytes())
```

Pipe:
```
	r, w := io.Pipe()
	go func() {
		defer w.Close()
		time.Sleep(3 * time.Second)
		w.Write([]byte("hello pipe"))
	}()

	io.Copy(os.Stdout, r)
```

Read All:
```
	r := strings.NewReader("hello world")
	b, err := io.ReadAll(r)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
```

Tee reader:
```
	r := strings.NewReader("hello world")
	tr := io.TeeReader(r, os.Stdout)
	if _, err := io.ReadAll(tr); err != nil {
		panic(err)
	}
```

# log
https://pkg.go.dev/log


Simple println with default logger:
```
	log.Println("hello world")
```

Custom logger:
```
	logger := log.New(os.Stdout, "mylogger: ", log.Lshortfile|log.LstdFlags)
	logger.Println("hello world")
```

# log/slog
https://pkg.go.dev/log/slog


Default logger info:
```
	slog.Info("hello", "count", 3)
```

Json output:
```
	logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
	logger.Info("hello", "count", 3)
```

With:
```
	logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
	logger = logger.With("url", "https://mywebsite.com")
	logger.Info("hello", "count", 3)
```

Group:
```
	logger := slog.Default().With("id", 42)
	parserLogger := logger.WithGroup("parser")
	parserLogger.Info("hello", "id", 43)
```

# maps
https://pkg.go.dev/maps

Clone:
```
	m := map[string]int{"answer": 42}
	m2 := maps.Clone(m)
	fmt.Println(m2)
```

Copy (appends or overrides existing key/value pairs):
```
	m := map[string]int{"answer": 42}
	m2 := map[string]int{"answer": 0}
	maps.Copy(m2, m)
```

Equal:
```
	m := map[string]int{"answer": 42}
	m2 := map[string]int{"answer": 0}
	eq := maps.Equal(m, m2)
	fmt.Println(eq)
```

Delete func:
```
	m := map[string]int{"answer": 42, "answer2": 0}
	maps.DeleteFunc(m, func(k string, v int) bool {
		return v == 0
	})
	fmt.Println(m)
```

# math/rand/v2
https://pkg.go.dev/math/rand/v2


Rand int:
```
	fmt.Println(rand.IntN(10))
```

Shuffle slice:
```
	words := []string{"a", "b", "c"}
	rand.Shuffle(len(words), func(i, j int) {
		words[i], words[j] = words[j], words[i]
	})
	fmt.Println(words)
```

# net

Listener:
```
func main() {
	ln, err := net.Listen("tcp", "localhost:8080")
	if err != nil {
		panic(err)
	}
	for {
		conn, err := ln.Accept()
		if err != nil {
			panic(err)
		}
		go handleConn(conn)
	}
}

func handleConn(conn net.Conn) {
	buf := make([]byte, 4)
	for {
		reqLen, err := conn.Read(buf)
		if err != nil {
			if err == io.EOF {
				fmt.Println("end of msg")
				break
			}
			panic(err)
		}
		fmt.Println("chunk of msg received ", string(buf[:reqLen]))
	}
}

```

Dialer:
```
func main() {
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		panic(err)
	}
	fmt.Fprintf(conn, "hello friend")
}
```

# net/http
https://pkg.go.dev/net/http


Make http requests using a new client:
```
	tr := &http.Transport{
	MaxIdleConns:       10,
	IdleConnTimeout:    30 * time.Second,
	DisableCompression: true,
	}

	client := &http.Client{Transport: tr}
	req, err := http.NewRequest("GET", "https://example.com", nil)
	if err != nil {
		panic(err)
	}
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	}
	fmt.Println(resp)
```

Handle func:
```
	http.HandleFunc("/foo", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "hello world")
	})
	log.Fatal(http.ListenAndServe(":8080", nil))
```

# os
https://pkg.go.dev/os

Get env:
```
	res := os.Getenv("PATH")
	fmt.Println(res)
```

Read file:
```
	data, err := os.ReadFile("f.txt")
	if err != nil {
		panic(err)
	}
	fmt.Println(string(data))
```

Read dir:
```
	files, err := os.ReadDir(".")
	if err != nil {
		panic(err)
	}
	for _, f := range files {
		fmt.Println(f.Name())
	}
```

Write file:
```
	err := os.WriteFile("f.txt", []byte("hello golang"), 0666)
	if err != nil {
		panic(err)
	}
```

# os/exec
https://pkg.go.dev/os/exec

Exec command and print output:
```
	out, err := exec.Command("ls", "-la").Output()
	if err != nil {
		panic(err)
	}
	fmt.Println(string(out))
```

# os/signal
https://pkg.go.dev/os/signal

Wait for interrupt signal:
```
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	s := <-c
	fmt.Println("signal received ", s)
```

# path
https://pkg.go.dev/path

Join
```
	u := path.Join("google.com", "foo")
	fmt.Println(u)
```

# path/filepath
https://pkg.go.dev/path/filepath

```
	ext := filepath.Ext("index.html")
	fmt.Println(ext)
```

# reflect
https://pkg.go.dev/reflect

Type of:
```
	var a int32 = 42
	fmt.Println(reflect.TypeOf(a))
```

# regexp
https://pkg.go.dev/regexp

Match string, find string:
```
	var re = regexp.MustCompile("foo.?")
	fmt.Println(re.MatchString("food"))
	fmt.Println(re.FindString("fool"))
```

# slices
https://pkg.go.dev/slices

More:
```
	s := []int{99, 1, 2, 3}
	fmt.Println(slices.Contains(s, 2))
	s2 := slices.Clone(s)
	fmt.Println(s2)
	slices.Sort(s)
	fmt.Println(s)
```

# sort
https://pkg.go.dev/sort

```
type Person struct {
	Salary int
	Name   string
}

func main() {
	staff := []Person{
		{Salary: 22, Name: "John"},
		{Salary: 42, Name: "Xavier"},
		{Salary: 11, Name: "Amos"},
	}
	sort.Slice(staff, func(i, j int) bool {
		return staff[i].Salary < staff[j].Salary
	})
	fmt.Println(staff)
}
```

# strconv
https://pkg.go.dev/strconv

String to int:
```
	i, err := strconv.Atoi("-42")
	if err != nil {
		panic(err)
	}
	fmt.Println(i, reflect.TypeOf(i))
```

Int to string

```
	i := 10
	s := strconv.Itoa(i)
	fmt.Printf("%T, %v\n", s, s)
```

Parse float (string to float):
```
	fString := "3.14"
	f, err := strconv.ParseFloat(fString, 64)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%T %v\n", f, f)
```

Format float (float to string):
```
	f := math.Pi
	fString := strconv.FormatFloat(f, 'f', -1, 64)
	fmt.Printf("%T, %s\n", fString, fString)
```

Append converted float string to byte slice:
```
	b := []byte("f32:")
	b = strconv.AppendFloat(b, math.Pi, 'f', -1, 64)
	fmt.Println(string(b))
```

Quote string:
```
	s := strconv.Quote(`"To Do" list ☺`)
	fmt.Println(s)
```

Quote string to ASCII:
```
	s := strconv.QuoteToASCII(`"To Do" list ☺`)
	fmt.Println(s)
```

# strings
https://pkg.go.dev/strings


Builder:
```
	var b strings.Builder
	for i := 0; i < 3; i++ {
		b.WriteString(fmt.Sprintf("hey %d, ", i))
	}
	fmt.Println(b.String())
```

Contains:
```
	s := "hello world"
	fmt.Println(strings.Contains(s, "ello"))
```

Replace:
```
	s := "hello world"
	fmt.Println(strings.Replace(s, "world", "golang", -1))
```

To upper:
```
	s := "hello world"
	fmt.Println(strings.ToUpper(s))
```

Trim space:
```
	s := "   hello world   "
	fmt.Printf("'%s'\n", strings.TrimSpace(s))
```

# sync
https://pkg.go.dev/sync

Wait group:
```
	wg := sync.WaitGroup{}
	wg.Add(1)
	go func() {
		defer wg.Done()
		fmt.Println("goroutine")
	}()
	wg.Wait()
	fmt.Println("main goroutine done")
```

RWMutex (anonymous field):
```
type BankAccount struct {
	sync.RWMutex
	Balance int64
}

func (b *BankAccount) Withdraw(amount int64) {
	b.RWMutex.Lock()
	b.Balance += amount
	b.RWMutex.Unlock()
}

func (b *BankAccount) PrintBalance() {
	b.RWMutex.RLock()
	fmt.Println(b.Balance)
	b.RWMutex.RUnlock()
}
```

# test
https://pkg.go.dev/testing

* Test:
```
//add_test.go
func TestAdd(t *testing.T) {
	got := Add(1, 2)
	const expect = 3
	if got != 3 {
		t.Errorf("expected %d, got %d", expect, got)
	}
}

```

* Subtests:
```
// add_test.go
func TestAdd(t *testing.T) {
	t.Run("case 1", func(t *testing.T) {
		got := Add(1, 2)
		const expect = 3
		if got != expect {
			t.Errorf("expected %d, got %d", expect, got)
		}

	})
	t.Run("case 2", func(t *testing.T) {
		got := Add(2, 2)
		const expect = 4
		if got != expect {
			t.Errorf("expected %d, got %d", expect, got)
		}

	})
}

```

* Parallel tests:
```
// add_test.go
func TestAdd(t *testing.T) {
	t.Parallel()
	//test code...
}

func TestAddMultiply(t *testing.T) {
	t.Parallel()
	//test code...
}

```

* Benchmark:
```
// add_test.go
func BenchmarkAdd(b *testing.B) {
	for range b.N {
		Add(1, 2)
	}
}

```

# time
https://pkg.go.dev/time


After:
```
	c := make(chan int)
	select {
	case v := <-c:
		fmt.Println(v)
	case <-time.After(5 * time.Second):
		fmt.Println("timeout!!")
	}
```

Subtract time:
```
	t0 := time.Now()
	time.Sleep(1 * time.Second)
	t1 := time.Now()
	fmt.Println("elapsed ", t1.Sub(t0))
```

Add time:
```
	t := time.Now()
	t = t.Add(time.Hour * 2)
```

Tick
```
	c := time.Tick(time.Second)
	for next := range c {
		fmt.Println(next)
	}
```

Sleep:
```
	time.Sleep(500 * time.Millisecond)
```

Unix time:
```
	fmt.Println(time.Now().Unix())
```

# unicode
https://pkg.go.dev/unicode

Is digit:
```
	fmt.Println(unicode.IsDigit('4'))
```

# unicode/utf8
https://pkg.go.dev/unicode/utf8

Rune count:
```
	str := "Hello, 世界"
	fmt.Println("bytes ", len(str))
	fmt.Println("runes ", utf8.RuneCountInString(str))
```

Valid string:
```
	valid := "Hello, 世界"
	invalid := string([]byte{0xff, 0xfe, 0xfd})
	fmt.Println(utf8.ValidString(valid))
	fmt.Println(utf8.ValidString(invalid))
```

Rune len:
```
	fmt.Println(utf8.RuneLen('界'))
```