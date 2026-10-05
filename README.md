# etax-go

> **Status: pre-alpha — under active development. APIs will change until v0.1.0.**
> See [PLAN.md](PLAN.md) for the roadmap.

Build, validate and sign Thai e-Tax Invoice & e-Receipt XML (ETDA standard) in Go.

สร้าง ตรวจสอบ และลงลายมือชื่อดิจิทัลเอกสาร e-Tax Invoice & e-Receipt ตามมาตรฐาน สพธอ. ด้วย Go

## Install
```sh
go get github.com/Chawn/etax-go
```

## Usage (target API)
```go
doc, err := etax.NewTaxInvoice(etax.T02).ID("INV-0001")./* ... */Build()
xmlBytes, _ := doc.MarshalXML()
```

## Development
```sh
go vet ./...
go test -race ./...
golangci-lint run   # if installed
```

## Contributing
Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Thai or English.

## License
MIT © Chawn and contributors
