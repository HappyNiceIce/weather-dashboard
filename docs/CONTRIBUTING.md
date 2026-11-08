\# Правила роботи з репозиторієм



\## Назви гілок

Формат: `<type>/<issue-id>-<short-name>`



Типи: `feature`, `fix`, `docs`, `refactor`, `chore`.



Приклади: `feature/1-show-sensor-data`, `docs/3-contributing`.



\## Формат commit

`<type>(<scope>): <короткий опис>`



Типи: feat, fix, docs, refactor, test, chore, build, ci.



Приклад: `docs(workflow): додано правила роботи з репозиторієм`



Кожен commit має містити одну логічну зміну.



\## Pull Request

1\. Прямі зміни в `main` заборонені.

2\. Робоча гілка створюється від актуальної `main`.

3\. В описі PR вказується, що зроблено і як перевірити.

4\. PR пов'язується з Issue рядком `Closes #<номер>`.



\## Зв'язок Issue → Branch → Pull Request

Кожна задача має Issue. Назва гілки містить номер Issue. PR закриває Issue після merge.



\## Review і merge

1\. Картка Issue переходить у статус Review.

2\. Перевіряється вкладка Files changed.

3\. Після перевірки виконується Merge pull request.

4\. Issue закривається, картка переходить у Done.

5\. Робоча гілка видаляється.

