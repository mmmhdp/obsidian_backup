# Basic staff
**Представим вот такой класс для генерации чисел фибоначи и сделаем пару базовых тестов для него**
```cpp
#include <vector>

class FibGen {
public:
  FibGen() = default;
  std::vector<int> GenerateNitems(std::size_t nitems) const;
};
```

```cpp
#include "rqtree.hpp"

#include <vector>

std::vector<int> FibGen::GenerateNitems(std::size_t nitems) const {
  std::vector<int> fib_seq;
  if (nitems == 0)
    return fib_seq;

  int left = 0;
  fib_seq.push_back(left);
  if (nitems == 1)
    return fib_seq;

  int right = 1;
  fib_seq.push_back(right);
  if (nitems == 2)
    return fib_seq;

  int next;
  while (nitems - 2 > 0) {
    next = left + right;
    fib_seq.push_back(next);

    left = right;
    right = next;

    --nitems;
  }

  return fib_seq;
}

```

```cpp
#include <gmock/gmock.h>
#include <gtest/gtest.h>

#include "rqtree.hpp"

class FigGenFixture : public testing::Test {
protected:
  FibGen g_ = FibGen();
};

/*
Конструктор и деструктор для работы с объектами, которые используются 
при работе набора (suite) тестов.
Важно знать, что когда-то нужно из заменять override'ом 
функций SetUp() и TearDown(), но вот когда - это уже в их доке расписано
*/

TEST_F(FigGenFixture, ZeroInput) {
  std::vector<int> ans{}, uans;
  int nitems = ans.size();

  uans = g_.GenerateNitems(nitems);

  EXPECT_TRUE(ans.size() == uans.size())
      << "Generated sequence lenght isn't correct";

  for (int i{0}; i != nitems; ++i)
    EXPECT_FALSE(ans[i] != uans[i])
        << "Incorrect value on position " << i << std::endl
        << "Correct: " << ans[i] << std::endl
        << "Provided: " << uans[i];
}


/*
Есть два распространнёных типа префиксов для ассертов:
EXPECT_* и ASSERT_*.
EXPECT_* - когда хотим и дальше собирать ошибки, т.е. 
провал теста плох, но мы бы хотели продолжить искать, чтобы найти 
ещё баги и фиксить их все вместе.

ASSERT_* - провал утверждение критичен. Например такой ассерт логичен 
перед разименованием указателя, который может быть nullptr, а
значит приведет к segfault, т.е. вообще говоря, при провале проверки 
такого указателя на nullptr дальше продолжать тестирование смысла нет.
*/

/*
Своё сообщние можно вот так интересно направить оператором <<, его 
дополнительно присовокупят к сгенерированному сообщению от самого фреймворка
*/

TEST_F(FigGenFixture, HandlesSmallPositiveInput) {
  std::vector<int> ans{0, 1}, uans;
  int nitems = ans.size();

  uans = g_.GenerateNitems(nitems);

  EXPECT_TRUE(ans.size() == uans.size())
      << "Generated sequence lenght isn't correct";

  for (int i{0}; i != nitems; ++i)
    EXPECT_FALSE(ans[i] != uans[i])
        << "Incorrect value on position " << i << std::endl
        << "Correct: " << ans[i] << std::endl
        << "Provided: " << uans[i];
}

TEST_F(FigGenFixture, HandlesMediumPositiveInput) {
  std::vector<int> ans{0, 1, 1, 2, 3, 5, 8, 13, 21, 34}, uans;
  int nitems = ans.size();

  uans = g_.GenerateNitems(nitems);

  EXPECT_TRUE(ans.size() == uans.size())
      << "Generated sequence lenght isn't correct";

  for (int i{0}; i != nitems; ++i)
    EXPECT_FALSE(ans[i] != uans[i])
        << "Incorrect value on position " << i << std::endl
        << "Correct: " << ans[i] << std::endl
        << "Provided: " << uans[i];
}

TEST_F(FigGenFixture, HandlesMediumInputDifferently) {
  std::vector<int> ans{0, 1, 1, 2}, uans;
  int nitems = ans.size();

  uans = g_.GenerateNitems(nitems);
  EXPECT_THAT(ans, testing::Eq(uans));
}

/*
Интересно, что лепить свою диагностику можно, но базовые контейнеры уже 
неплохо обложены необходимым для диагностики, что позволяет 
просто пихать их в подобные ассерты.
*/
```