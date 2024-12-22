:::info
This is NOT a component library. It's a collection of re-usable components that you can copy and paste into your apps.

:::

  
[shadcn/ui](https://ui.shadcn.com/) 和传统 UI 组件库最大的区别是 shadcn/ui 不通过 npm 分发使用，而是创建源码文件到你的项目本地，然后自由修改使用。正如官网中解释的那样：Pick the components you need. Copy and paste the code into your project and customize to your needs.

## 复制源代码的好处
相对于调用组件渲染 UI，将源代码复制到本地修改有几个明显的好处

+ 有了源码后可以根据项目需求自由修改组件的每一个细节，不受原有组件库的限制，尤其是不再需要使用各种 hack 手段覆盖组件内部样式
+ 可以删除项目中不需要的代码，减少冗余，能够根据具体使用场景进行针对性的性能优化
+ 当遇到问题时可以直接在源码层面进行调试，而不是依赖于黑盒组件
+ 不需要担心第三方库的版本更新可能带来的兼容性问题，减少了项目的外部依赖，提高了项目的稳定性和可控性。同时也不依赖于第三方库的更新和维护，可以确保项目长期的可维护性

总而言之 shadcn/ui 虽然需要更多的初始工作和维护努力，但它提供了更大的灵活性和控制力，特别适合那些需要高度定制化的项目

## 安装
按照官方的 [installation](https://ui.shadcn.com/docs/installation) 文档安装好依赖，就可以添加组件了。等等！不是说不通过 npm 分发组件吗？怎么还需要安装依赖？？？

因为 shadcn/ui 底层大量使用了 [Radix UI ](https://www.radix-ui.com/)的无样式组件作为基础，生成的代码使用了 TailwindCSS 的类名，不安装配置 TailwindCSS 组件没有对应的默认样式

如果希望在页面使用一个 Card 组件，在项目执行命令 `npx shadcn@latest add card`，这时候项目会新增 `src/components/ui/card.tsx` 文件，内部实现了 Card 组件，我们在项目中可以直接使用

```jsx
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card"

<Card>
  <CardHeader>
    <CardTitle>Card Title</CardTitle>
    <CardDescription>Card Description</CardDescription>
  </CardHeader>
  <CardContent>
    <p>Card Content</p>
  </CardContent>
  <CardFooter>
    <p>Card Footer</p>
  </CardFooter>
</Card>
```

用同样的方法简单加点料，就可以做出一个精美的表单了

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732421929397-4dee50c9-47ce-4440-bc49-3f92fded92f8.png)

```jsx
npx shadcn@latest add button card input label select
```

```jsx
import * as React from "react"

import { Button } from "@/components/ui/button"
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select"

export function CardWithForm() {
  return (
    <Card className="w-[350px]">
      <CardHeader>
        <CardTitle>Create project</CardTitle>
        <CardDescription>Deploy your new project in one-click.</CardDescription>
      </CardHeader>
      <CardContent>
        <form>
          <div className="grid w-full items-center gap-4">
            <div className="flex flex-col space-y-1.5">
              <Label htmlFor="name">Name</Label>
              <Input id="name" placeholder="Name of your project" />
            </div>
            <div className="flex flex-col space-y-1.5">
              <Label htmlFor="framework">Framework</Label>
              <Select>
                <SelectTrigger id="framework">
                  <SelectValue placeholder="Select" />
                </SelectTrigger>
                <SelectContent position="popper">
                  <SelectItem value="next">Next.js</SelectItem>
                  <SelectItem value="sveltekit">SvelteKit</SelectItem>
                  <SelectItem value="astro">Astro</SelectItem>
                  <SelectItem value="nuxt">Nuxt.js</SelectItem>
                </SelectContent>
              </Select>
            </div>
          </div>
        </form>
      </CardContent>
      <CardFooter className="flex justify-between">
        <Button variant="outline">Cancel</Button>
        <Button>Deploy</Button>
      </CardFooter>
    </Card>
  )
}
```

## 轮播 demo
shadcn/ui 的核心理念是不需要作为一个单一的、完整的组件库来安装。但这并不意味着它的所有组件都是完全零依赖的，某些复杂的组件如轮播等需要额外的库来实现其核心功能

当需要使用特定的组件时，shadcn/ui 会告诉你需要安装哪些额外的依赖。这样只需为实际使用的组件安装必要的依赖，而不是安装一个包含所有可能依赖的大型库

轮播（Carousel）底层依赖 [Embla Carousel](https://www.embla-carousel.com/)<font style="color:rgb(9, 9, 11);">，通过 cli 命令仍然安逸一键生成源码，自动安装必要的依赖。手工安装时候 shadcn/ui 会给组件的源码及其必要的依赖</font>

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732422457324-c2669927-edb1-47eb-9606-52abc25e5e79.png)

使用同样非常简单

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732422508284-33f86dae-c6de-4743-9ae1-a8cd4d90ff6d.png)

```jsx
import * as React from "react"

import { Card, CardContent } from "@/components/ui/card"
import {
  Carousel,
  CarouselContent,
  CarouselItem,
  CarouselNext,
  CarouselPrevious,
} from "@/components/ui/carousel"

export function CarouselSize() {
  return (
    <Carousel
      opts={{
        align: "start",
      }}
      className="w-full max-w-sm"
    >
      <CarouselContent>
        {Array.from({ length: 5 }).map((_, index) => (
          <CarouselItem key={index} className="md:basis-1/2 lg:basis-1/3">
            <div className="p-1">
              <Card>
                <CardContent className="flex aspect-square items-center justify-center p-6">
                  <span className="text-3xl font-semibold">{index + 1}</span>
                </CardContent>
              </Card>
            </div>
          </CarouselItem>
        ))}
      </CarouselContent>
      <CarouselPrevious />
      <CarouselNext />
    </Carousel>
  )
}
```

  


