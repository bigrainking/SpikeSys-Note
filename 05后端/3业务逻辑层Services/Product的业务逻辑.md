

[toc]



# 一、Product的相关业务

- 业务逻辑层：处理从Controller发来的业务逻辑内容，调用Model层repository获取数据库查询封装后的结果

```go
package services

import (
	"Spike-Product/datamodels"
	"Spike-Product/repositories"
)

// 业务逻辑层：对接业务
type IProductService interface {
	// 获取产品
	GetProductByID(ID int64) (product *datamodels.Product, err error)
	// 获取所有产品
	GetAllProduct() (allProducts []*datamodels.Product, err error)
	// 增删改
	InsertProduct(product *datamodels.Product) (ID int64, err error)
	DeleteProductByID(ID int64) (success bool, err error)
	UpdateProduct(product *datamodels.Product) error
}

// 实现接口
type IProductServiceManager struct {
	repositoriesProduct repositories.IProduct //为了调用repositories中所有的函数
}

// 初始化函数
func NewIProductServiceManager(repositoriesProduct repositories.IProduct) IProductService {
	return &IProductServiceManager{repositoriesProduct}
}

// 获取产品
func (p *IProductServiceManager) GetProductByID(ID int64) (product *datamodels.Product, err error) {
	return p.repositoriesProduct.SelectById(ID)
}

// 获取所有产品
func (p *IProductServiceManager) GetAllProduct() (allProducts []*datamodels.Product, err error) {
	return p.repositoriesProduct.SelectAll()
}

// 增删改
func (p *IProductServiceManager) InsertProduct(product *datamodels.Product) (ID int64, err error) {
	return p.repositoriesProduct.Insert(product)
}
func (p *IProductServiceManager) DeleteProductByID(ID int64) (success bool, err error) {
	return p.repositoriesProduct.Delete(ID)
}
func (p *IProductServiceManager) UpdateProduct(product *datamodels.Product) error {
	return p.repositoriesProduct.Update(product)
}
```

