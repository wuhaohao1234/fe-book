![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732445244578-5bfc4807-168d-47ea-a8a8-255e21eaee43.png)

## 最简单的实现
```jsx
import React, { useEffect, useState } from 'react';

interface ICarouselProps {
  images: string[];
}

const Carousel: React.FC<ICarouselProps> = ({ images }) => {
  const [currentIndex, setCurrentIndex] = useState<number>(0);
  const len = images.length;

  const nextSlide = () => {
    setCurrentIndex(prev => (prev + 1) % len);
  }

  const prevSlide = () => {
    setCurrentIndex(prev => (prev - 1 + len) % len);
  };

  return (
    <div className="relative w-full max-w-3xl mx-auto">
      <div className="overflow-hidden relative h-64">
        {
          images.map((image, index) => (
            <div
              key={index}
              className={`absolute top-0 left-0 w-full h-full transition-opacity duration-500 ease-in-out ${
                index === currentIndex ? 'opacity-100' : 'opacity-0'
              }`}
            >
              <img src={image} className="w-full h-full object-cover"/>
            </div>
          ))
        }
      </div>
      <button
        onClick={prevSlide}
        className="absolute top-1/2 left-4 transform -translate-y-1/2 bg-black bg-opacity-50 text-white p-2 rounded-full"
      >
        &#10094;
      </button>
      <button
        onClick={nextSlide}
        className="absolute top-1/2 right-4 transform -translate-y-1/2 bg-black bg-opacity-50 text-white p-2 rounded-full"
      >
        &#10095;
      </button>
    </div>
  );
}

export default Carousel;
```

为了避免定时器因为闭包保留了之前的 currentIndex，`nextSlide`和 `prevSlide`内部要使用函数式参数调用 `setCurrentIndex`

## 加一下自动轮播
首先需要支持自动轮播的间隔

```jsx
interface ICarouselProps {
  images: string[];
  interval?: number;
}
```

在 useEffect 中定时调用 `nextSlide`，并在函数重新调用时候清理定时器

```jsx
const Carousel: React.FC<ICarouselProps> = ({ images, interval = 5000 }) => {
  useEffect(() => {
      const timer = setInterval(nextSlide, interval);
      return () => {
        clearInterval(timer);
      }
    }, [])

  return (/* ... */);
}
```

可以在图片下面放个当前 index 的指示器

![](https://cdn.nlark.com/yuque/0/2024/png/87727/1732442131560-08fd0878-18a7-4780-8d82-eda7642158de.png)

```jsx
<div className="absolute bottom-4 left-1/2 transform -translate-x-1/2 flex space-x-2">
  {images.map((_, index) => (
    <button
      key={index}
      onClick={() => setCurrentIndex(index)}
      className={`w-3 h-3 rounded-full ${
        index === currentIndex ? 'bg-white' : 'bg-gray-400'
      }`}
    ></button>
  ))}
</div>
```

## 手工切换 slide 后定时轮播重新计时
这时候需要使用保留一个全局的 timerRef，每次调用 prevSlide 或 nextSlide 时候清理 timer

```tsx
const Carousel: React.FC<ICarouselProps> = ({ images, interval = 5000 }) => {
  const [currentIndex, setCurrentIndex] = useState<number>(0);
  const len = useMemo(() => images.length, []);

  type TimerType = ReturnType<typeof setInterval> | null;
  const timerRef = useRef<TimerType>(null);

  const resetTimer = useCallback(() => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
    }
    timerRef.current = setInterval(nextSlide, interval);
  }, [interval]);

  const nextSlide = useCallback(() => {
    setCurrentIndex(prev => (prev + 1) % len);
    resetTimer();
  }, [len, resetTimer]);

  const prevSlide = useCallback(() => {
    setCurrentIndex(prev => (prev - 1 + len) % len);
    resetTimer();
  }, [len, resetTimer]);

  const goToSlide = useCallback((index: number) => {
    setCurrentIndex(index);
    resetTimer();
  }, [resetTimer]);

  useEffect(() => {
    resetTimer();
    return () => {
      if (timerRef.current != null) {
        clearInterval(timerRef.current);
      }
    }
  }, [resetTimer]);

  return (

  );
}
```

