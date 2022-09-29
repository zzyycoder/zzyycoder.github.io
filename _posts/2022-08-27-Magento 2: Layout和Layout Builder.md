---
title: 'Magento 2: Layout和Layout Builder'
date: 2022-08-27
categories: []
tags: []
mermaid: true
img_path: /assets/img
---
![Layout and its builder](layout-and-its-builder.jpg)
#### NOTES:
1. `Builder` 在 `__construct` 方法中设置了 `Layout` 的 `builder` 属性。
    ```php
    public function __construct(
        View\LayoutInterface $layout,
        App\Request\Http $request,
        Event\ManagerInterface $eventManager
    ) {
        $this->layout = $layout;
        $this->request = $request;
        $this->eventManager = $eventManager;
        $this->layout->setBuilder($this); // 这里设置了layout的builder属性
    }
    ```
2. 由客户端 `Result\Layout` 决定 `Builder` 的类型。
    ```php
    public function __construct(
           View\Element\Template\Context $context,
           View\LayoutFactory $layoutFactory,
           View\Layout\ReaderPool $layoutReaderPool,
           Framework\Translate\InlineInterface $translateInline,
           View\Layout\BuilderFactory $layoutBuilderFactory,
           View\Layout\GeneratorPool $generatorPool,
           $isIsolated = false
       ) {
           $this->layoutFactory = $layoutFactory;
           $this->layoutBuilderFactory = $layoutBuilderFactory;
           $this->layoutReaderPool = $layoutReaderPool;
           $this->eventManager = $context->getEventManager();
           $this->request = $context->getRequest();
           $this->translateInline = $translateInline;
           // TODO Shared layout object will be deleted in MAGETWO-28359
           $this->layout = $isIsolated
               ? $this->layoutFactory->create(['reader' => $this->layoutReaderPool, 'generatorPool' => $generatorPool])
               : $context->getLayout();
           $this->layout->setGeneratorPool($generatorPool);
           $this->initLayoutBuilder(); // 在这里创建具体的builder类型
       }
    ```
