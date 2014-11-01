//before running make sure you setup a GOPATH env variable and ran: "go get code.google.com/p/go.net/html"
 
//to run: go run ioCrawler.go -url="http://developers.google.com/"
//also try http://developer.android.com/index.html
//output goo.gl links to try and redeem will be sent to foundLinks.txt
 
//by the way there's an artificial "rate limit" in func crawler, you can lower that or raise it as you wish
//You can also comment out the onlyGoogleRegex code if you don't want to limit to google.com/youtube
//if you're getting I/O timeout errors, then you might need to increase the timeouts on line 231
 
package main
 
import (
	"code.google.com/p/go.net/html"
	"errors"
	"flag"
	"fmt"
	"io/ioutil"
	"net"
	"net/http"
	"net/url"
	"os"
	"os/signal"
	"regexp"
	"strings"
	"time"
	"sync"
	"math/rand"
)
 
//prevent from downloading files we can't even read
var extToIgnore = regexp.MustCompile(`(?i)\.(gz|tar|tgz|bz2|jpg|png|jpeg|gif|psd|mp3|webm|ogg|mp4|mpeg|flv|avi|wmv|fla|mov|bin|wav|rpm|dmg|exe|zip|rar|7z|doc|ppt|pdf|iso|torrent)$`)
 
//if you want to index more domains feel free to update this or remove it entirely (update validateUri too to remove the check at the bottom)
var onlyGoogleRegex = regexp.MustCompile(`(?i)(google|youtube|android|golang|opers.blogspot|oogle.blogspot|youtu|angularjs|angulardart|dartlang|html5rocks)\.(com|org|net|io|be)`)
 
var googlRegex = regexp.MustCompile(`(?im)((?:goo\.gl/[a-zA-Z0-9]{3,9})|(?:[a-zA-Z0-9]{3,9}(?:/|\\)lg\.oog))`)
 
//if you're getting a lot of I/O (pun not intended...) timeouts then raise this
var defaultConnectTimeout = time.Duration(2*time.Second)
var defaultRWTimeout = time.Duration(5*time.Second)
 
var guessTimeout = time.Duration(1000*time.Millisecond)
var rateLimitedGoog bool
 
var queuedLock = sync.RWMutex{}
var knownLock = sync.RWMutex{}
var knownGooglLock = sync.RWMutex{}
var queuedGooglLock = sync.RWMutex{}
 
var queuedUris = map[string]bool{}
var knownUris = map[string]bool{}
var queuedGoogl = map[string]bool{}
var knownGoogl = map[string]bool{
	"goo.gl/ZV63Cr": true,
	"goo.gl/1nToQ4": true,
	"goo.gl/q7wKp8": true,
	"goo.gl/dDZIsO": true,
	"goo.gl/R1jVTj": true,
	"goo.gl/B92rvj": true,
	"goo.gl/Au49Um": true,
	"goo.gl/uQvdDQ": true,
	"goo.gl/KBXDW9": true,
	"goo.gl/m3uwvU": true,
	"goo.gl/bhLDq0": true,
	"goo.gl/gdEknO": true,
	"goo.gl/8AUTRP": true,
	"goo.gl/5nz6cg": true,
	"goo.gl/T3mzjk": true,
	"goo.gl/5FwCJy": true,
	"goo.gl/3QGGK0": true,
	"goo.gl/K6E7l8": true,
	"goo.gl/xZM9jt": true,
	"goo.gl/rMFYNX": true,
	"goo.gl/F3noUV": true,
	"goo.gl/JYSlLQ": true,
	"goo.gl/hmgbrW": true,
	"goo.gl/0gNnSE": true,
	"goo.gl/Tw6kP9": true,
	"goo.gl/RB2zhx": true,
	"goo.gl/kJPqT3": true,
	"goo.gl/qllLsG": true,
	"goo.gl/zj9bO0": true,
	"goo.gl/bjAPLg": true,
	"goo.gl/7tKTyd": true,
	"goo.gl/eXeVao": true,
	"goo.gl/Dpoyqi": true,
	"goo.gl/iztHMd": true,
	"goo.gl/HACUad": true,
	"goo.gl/oJDnYs": true,
	"goo.gl/ia7jLd": true,
	"goo.gl/AJRPcI": true,
	"goo.gl/bS7lhD": true,
	"goo.gl/lyBKa0": true,
	"goo.gl/OmM6DN": true,
	"goo.gl/puUsyX": true,
	"goo.gl/Gc8Dsd": true,
	"goo.gl/P3cEUu": true,
	"goo.gl/9lT4ZV": true,
	"goo.gl/HNA9YG": true,
	"goo.gl/xjP4Ke": true,
	"goo.gl/ykmzUC": true,
	"goo.gl/EOiis9": true,
	"goo.gl/wgzU7N": true,
	"goo.gl/5ixjt4": true,
	"goo.gl/L2N5sY": true,
	"goo.gl/SWNo61": true,
	"goo.gl/06Hov3": true,
	"goo.gl/JOzJ9j": true,
	"goo.gl/8JoXn0": true,
	"goo.gl/y1y9Lh": true,
	"goo.gl/FDTdEx": true,
	"goo.gl/LgqaL0": true,
	"goo.gl/OQwgds": true,
	"goo.gl/hZWUau": true,
	"goo.gl/wfkUr7": true,
	"goo.gl/sK6kAA": true,
	"goo.gl/DpYUwO": true,
	"goo.gl/vLppsl": true,
	"goo.gl/KVDTVy": true,
	"goo.gl/dGrMwY": true,
	"goo.gl/YOTeO1": true,
	"goo.gl/PltckZ": true,
	"goo.gl/sUJ1r7": true,
	"goo.gl/okXi4W": true,
	"goo.gl/ENWVvN": true,
	"goo.gl/BcLOM9": true,
	"goo.gl/R9MgF3": true,
	"goo.gl/egnhgc": true,
	"goo.gl/Jca6KP": true,
	"goo.gl/1AiXEk": true,
	"goo.gl/iMmkIj": true,
	"goo.gl/E1vTJk": true,
	"goo.gl/pikTTi": true,
	"goo.gl/9KK0k0": true,
	"goo.gl/Oof8mG": true,
	"goo.gl/X1R54k": true,
	"goo.gl/ykH2OP": true,
	"goo.gl/cuSgmK": true,
	"goo.gl/I6Lpbz": true,
	"goo.gl/JFKDGw": true,
	"goo.gl/o0YxL2": true,
	"goo.gl/cCpQrU": true,
	"goo.gl/sWHuKQ": true,
	"goo.gl/FhGPJW": true,
	"goo.gl/v6jFEL": true,
	"goo.gl/hBKyB5": true,
	"goo.gl/mH9rXs": true,
	"goo.gl/3QXZ9r": true,
	"goo.gl/6M31VD": true,
	"goo.gl/ZaQQyr": true,
	"goo.gl/FbPhW1": true,
	"goo.gl/EePxH9": true,
	"goo.gl/LLy4UO": true,
	"goo.gl/yGQsaV": true,
	"goo.gl/MtXamT": true,
	"goo.gl/SaKrk1": true,
	"goo.gl/Jtc6vr": true,
}
var uriQueue = make(chan *url.URL, 16384)
var googlQueue = make(chan *url.URL, 4096)
 
var googlOutputFile *os.File
var shouldGuess bool
 
func main() {
	startURL := flag.String("url", "", "the url to start crawling")
	guess := flag.Bool("guess", false, "guess goo.gl links")
	flag.Parse()
	shouldGuess = *guess
	startUri, err := url.Parse(*startURL)
	if err != nil || *startURL == "" {
		fmt.Printf("Failed to parse starting URL: %s. Response: %s\n", *startURL, err)
		return
	}
 
	googlOutputFile, err = os.OpenFile("./foundLinks.txt", os.O_APPEND | os.O_CREATE | os.O_WRONLY, 0644)
	if err != nil {
		fmt.Printf("Failed to open foundLinks.txt. Error: %s\n", err)
		return
	}
	defer func() {
		if err := googlOutputFile.Close(); err != nil {
			fmt.Printf("Failed to close output file: %s\n", err)
		}
	}()
 
	for i := 0; i < 48; i++ {
		go crawler()
	}
	for i := 0; i < 8; i++ {
		go googlChecker()
	}
	for i := 0; i < 1; i++ {
		go guesser()
	}
	//fakeGoogl, _ := url.Parse("http://goo.gl/ZV63Cr")
	//googlQueue <- fakeGoogl
	uriQueue <- startUri
	waitForSignal()
}
 
func crawler() {
	var nextUri *url.URL
	for {
		<-time.After(500*time.Millisecond)
		select {
		case nextUri = <-uriQueue:
			crawl(nextUri)
		}
	}
}
 
func googlChecker() {
	var nextUri *url.URL
	for {
		if rateLimitedGoog {
			fmt.Println("RATE LIMITED!! Waiting...")
			<-time.After(120*time.Second)
		}
		select {
		case nextUri = <-googlQueue:
			verifyGoogl(nextUri)
		}
	}
}
 
func normalizeUri(uri *url.URL) string {
	host := uri.Host
	//remove www from the host
	if len(host) > 4 && host[0:4] == "www." {
		host = host[4:]
	}
	path := uri.Path
	if path == "" || path == "/" {
		return host
	}
	pathLen := len(path)
	//remove any trailing slashes
	if path[pathLen-1] == '/' {
		path = path[:pathLen-1]
	}
	//unfortunately including RawQuery in here to fix youtube links not working anymore
	return strings.Join([]string{host, path, uri.RawQuery}, "")
}
 
 
func validateUri(uri *url.URL) (bool, string) {
	if uri.Scheme != "http" && uri.Scheme != "https" {
		return false, ""
	}
	normalUri := normalizeUri(uri)
	knownLock.RLock()
	if _, ok := knownUris[normalUri]; ok {
		knownLock.RUnlock()
		//already checked this
		return false, normalUri
	}
	knownLock.RUnlock()
	if uri.Host == "" {
		return false, normalUri
	}
	if extToIgnore.MatchString(uri.Path) {
		return false, normalUri
	}
	if !onlyGoogleRegex.MatchString(uri.Host) {
		return false, normalUri
	}
	return true, normalUri
}
 
//from http://stackoverflow.com/questions/16895294/how-to-set-timeout-for-http-get-requests-in-golang
func TimeoutDialer(cTimeout time.Duration, rwTimeout time.Duration) func(net, addr string) (c net.Conn, err error) {
	return func(netw, addr string) (net.Conn, error) {
		conn, err := net.DialTimeout(netw, addr, cTimeout)
		if err != nil {
			return nil, err
		}
		conn.SetDeadline(time.Now().Add(rwTimeout))
		return conn, nil
	}
}
func NewTimeoutClient(connectTimeout time.Duration, readWriteTimeout time.Duration, baseNormalUri string) *http.Client {
	return &http.Client{
		Transport: &http.Transport{
			Dial: TimeoutDialer(connectTimeout, readWriteTimeout),
		},
		CheckRedirect: func(req *http.Request, via []*http.Request) error {
			valid, newNormalUri := validateUri(req.URL)
			//allow redirects to the *same* url since we might already have this url in knownUris because we're processing it right now
			if !valid && baseNormalUri != newNormalUri {
				return errors.New("Redirect:Invalid")
			}
			return nil
		},
	}
}
 
func crawl(uri *url.URL) {
	valid, normalUri := validateUri(uri)
	if !valid || uri == nil {
		queuedLock.Lock()
		delete(queuedUris, normalUri)
		queuedLock.Unlock()
		return
	}
	knownLock.Lock()
	knownUris[normalUri] = true
	knownLock.Unlock()
 
	queuedLock.Lock()
	delete(queuedUris, normalUri)
	queuedLock.Unlock()
 
	findYoutubeAnnotations(uri)
 
	urlStr := uri.String()
	fmt.Printf("Fetching: %s\n", urlStr)
	client := NewTimeoutClient(defaultConnectTimeout, defaultRWTimeout, normalUri)
	resp, err := client.Get(urlStr)
	if err != nil {
		var errMsg string
		urlErr, ok := err.(*url.Error)
		if !ok {
			errMsg = err.Error()
		} else {
			errMsg = urlErr.Err.Error()
		}
 
		if errMsg != "Redirect:Invalid" {
			fmt.Printf("Failed to Get URL: %s. Response: %s\n", urlStr, errMsg)
		}
		knownLock.Lock()
		delete(knownUris, normalUri)
		knownLock.Unlock()
		return
	}
	htmlBytes, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("Failed to read from server to get full HTML for %s. Error: %s\n", urlStr, err)
		knownLock.Lock()
		delete(knownUris, normalUri)
		knownLock.Unlock()
		return
	}
	resp.Body.Close()
	htmlStr := string(htmlBytes)
	locateAndSaveGoogl(htmlStr)
 
	doc, err := html.Parse(strings.NewReader(htmlStr))
	if err != nil {
		fmt.Printf("Failed to parse HTML for %s. Error: %s\n", urlStr, err)
		return
	}
	findAnchors(doc, uri)
}
 
func findAnchors(node *html.Node, baseUri *url.URL) {
	if node.Type == html.ElementNode && (node.Data == "a" || node.Data == "url") {
		for _, attr := range node.Attr {
			if (node.Data == "a" && attr.Key == "href") || (node.Data == "url" && attr.Key == "value") {
				newUri, err := url.Parse(attr.Val)
				if err != nil || newUri == nil {
					fmt.Printf("Invalid url detected: %s. Error %s\n", attr.Val, err)
					break
				}
				if !newUri.IsAbs() {
					newUri = baseUri.ResolveReference(newUri)
				}
				valid, newNorm := validateUri(newUri)
				if valid {
					//fmt.Printf("Found new URL! %s\n", newUri.String())
					queuedLock.Lock()
					if _, ok := queuedUris[newNorm]; !ok {
						queuedUris[newNorm] = true
						queuedLock.Unlock()
 
						select {
						case uriQueue <- newUri:
						case <-time.After(time.Second * 1):
							//fmt.Println("Queue is full!!")
							return
						}
					} else {
						queuedLock.Unlock()
					}
				}
				break
			}
		}
	} else if node.Type == html.ElementNode && node.Data == "head" {
		//skip looping through head
		node = node.NextSibling
		if node == nil {
			return
		}
	}
	for nextNode := node.FirstChild; nextNode != nil; nextNode = nextNode.NextSibling {
		findAnchors(nextNode, baseUri)
	}
}
 
func findYoutubeAnnotations(uri *url.URL) {
	if (uri.Host == "youtube.com" || uri.Host == "www.youtube.com") {
		if len(uri.Path) < 6 || uri.Path[0:6] != "/watch" {
			return
		}
		queryVals := uri.Query();
		if queryVals == nil {
			return
		}
		videoIDs, ok := queryVals["v"]
		if !ok || len(videoIDs) < 1 {
			videoIDsCommas, ok := queryVals["video_ids"]
			if !ok || len(videoIDsCommas) < 1 {
				return
			}
			videoIDs = strings.Split(videoIDsCommas[0], ",")
		}
		for _, videoID := range videoIDs {
			addYoutubeAnnotations(videoID)
		}
	} else if (uri.Host == "youtu.be") {
		addYoutubeAnnotations(uri.Path);
	} else {
		return
	}
}
 
//thanks https://plus.google.com/+MarcLesterTan
func addYoutubeAnnotations(videoID string) {
	if len(videoID) < 5 {
		return
	}
	newUri, err := url.Parse("https://www.youtube.com/annotations_invideo?features=1&legacy=1&video_id=" + videoID)
	if err != nil || newUri == nil {
		fmt.Printf("Invalid youtube annotations url generated. Error %s\n", err)
		return
	}
	valid, newNorm := validateUri(newUri)
	if valid {
		//fmt.Printf("Added new youtube annotations URL! %s\n", newUri.String())
		queuedLock.Lock()
		if _, ok := queuedUris[newNorm]; !ok {
			queuedUris[newNorm] = true
			queuedLock.Unlock()
 
			select {
			case uriQueue <- newUri:
			case <-time.After(time.Second * 1):
				//fmt.Println("Queue is full!!")
				return
			}
		} else {
			queuedLock.Unlock()
		}
	}
}
 
func ReverseString(str string) string {
	i := len(str)
	final := make([]rune, i)
    for _, r := range str {
		i--
        final[i] = r
    }
    return string(final)
}
 
func locateAndSaveGoogl(htmlStr string) {
	matches := googlRegex.FindAllString(htmlStr, -1)
	if matches == nil {
		return
	}
	for _, match := range matches {
		//detect backwards goo.gl links
		if len(match) > 6 && match[len(match)-6:] == "lg.oog" {
			match = ReverseString(match)
		}
		//fmt.Printf("Found goo.gl link!: %s\n", match)
		//prevent backwards slashes from breaking this
		uri, err := url.Parse("http://" + strings.Replace(match, `\`, "/", -1))
		if err != nil {
			fmt.Printf("Failed to parse goo.gl link: %s. Error: %s\n", match, err)
			continue
		}
		normalUri := normalizeUri(uri)
		knownGooglLock.RLock()
		if _, ok := knownGoogl[normalUri]; ok {
			knownGooglLock.RUnlock()
			continue
		}
		knownGooglLock.RUnlock()
 
		queuedGooglLock.Lock()
		if _, ok := queuedGoogl[normalUri]; ok {
			queuedGooglLock.Unlock()
			continue
		}
		queuedGoogl[normalUri] = true
		queuedGooglLock.Unlock()
 
		googlQueue <- uri
	}
}
 
var validGooglChars = "avE1k4yiQhAzl9B6DpIRdCqnF8tbSr7J2NKGT3VmoMPuc5UHeLjOZWxfwXgsY0"
 
func guesser() { //s float64
	if !shouldGuess {
		return
	}
 
	var i int
	n := make([]uint8, 6)
	for {
		if rateLimitedGoog {
			<-time.After(180*time.Second)
			continue
		}
 
		i = 0
		for i < 6 {
			n[i] = validGooglChars[int(rand.Float64() * 62)]
			i++
		}
		normal := "goo.gl/" + string(n)
		knownGooglLock.RLock()
		if _, ok := knownGoogl[normal]; ok {
			knownGooglLock.RUnlock()
			continue
		}
		knownGooglLock.RUnlock()
 
		queuedGooglLock.Lock()
		if _, ok := queuedGoogl[normal]; ok {
			queuedGooglLock.Unlock()
			continue
		}
		queuedGoogl[normal] = true
		queuedGooglLock.Unlock()
 
		uri, err := url.Parse("http://" + normal)
		if err != nil {
			fmt.Printf("Failed to generate valid goo.gl link: %s. Error: %s\n", normal, err)
			continue
		}
		if guessTimeout > 0 {
			<-time.After(guessTimeout)
		}
		googlQueue <- uri
	}
}
 
var googlClient = &http.Client{
	CheckRedirect: func(req *http.Request, via []*http.Request) error {
		// /events/io/2014/redeem
		//yes... this is ghetto how it works
		if req.URL.Host == "developers.google.com" && len(req.URL.Path) > 22 && req.URL.Path[0:22] == "/events/io/2014/redeem" {
			return errors.New("Redirect:Valid")
		}
		return errors.New("Redirect:Invalid")
	},
}
 
func verifyGoogl(uri *url.URL) {
	normalUri := normalizeUri(uri)
 
	queuedGooglLock.Lock()
	delete(queuedGoogl, normalUri)
	queuedGooglLock.Unlock()
 
	knownGooglLock.Lock()
	if _, ok := knownGoogl[normalUri]; ok {
		knownGooglLock.Unlock()
		return
	}
	knownGoogl[normalUri] = true
	knownGooglLock.Unlock()
 
	urlStr := uri.String()
	fmt.Println("Fetching goo.gl link: " + urlStr)
	resp, err := googlClient.Head(urlStr)
	if err != nil {
		var errMsg string
		urlErr, ok := err.(*url.Error)
		if !ok {
			errMsg = err.Error()
		} else {
			errMsg = urlErr.Err.Error()
		}
 
		if errMsg == "Redirect:Valid" {
			rateLimitedGoog = false
			fmt.Printf("Found a valid goo.gl redirect!! %s\n", urlStr)
 
			if _, err := googlOutputFile.WriteString(urlStr + "\n"); err != nil {
				fmt.Printf("ERROR! Failed to write goo.gl redirect to output file! Error: %s/n", err)
			}
		} else if errMsg == "Redirect:Invalid" {
			rateLimitedGoog = false
			//do nothing
		} else {
			fmt.Printf("Failed to Head URL: %s. Response: %s\n", urlStr, errMsg)
		}
	}
	if resp.StatusCode == 403 {
		rateLimitedGoog = true
	} else {
		rateLimitedGoog = false
	}
}
 
func waitForSignal() {
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)
	<-c
	fmt.Println("Received SIGINT!")
}
