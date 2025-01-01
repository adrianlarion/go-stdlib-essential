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