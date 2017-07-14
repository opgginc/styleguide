# Introduce
## 이 문서는
- OP.GG 에서 Laravel 을 사용하는 프로젝트에 대해서 이야기한다.
- 문체는 [간결체](http://terms.naver.com/entry.nhn?docId=1055742&cid=40942&categoryId=32860)를 사용하도록 한다.

## 시스템 기준
- PHP 버전은 7.1 을 기준으로 설명한다. 
- Laravel 버전은 5.4 를 기준으로 설명한다.
- IDE는 PHPStorm 를 기준으로 설명한다.

## Laravel 로 프로젝트를 시작하기 전에 읽어야 할 글
- 공식 [Laravel 문서](https://laravel.com/docs/)
- [PHP The Right Way](http://modernpug.github.io/php-the-right-way/)

# Project Architecture
## Directory Structure
- routes
    - 모든 URL 들의 구성을 확인 할 수 있다. 여기에 없으면 없는 URL 이다.
- app/Http/Controllers
    - routes 와 직접적으로 연계된 컨트롤러이다. 모든 URL의 작동은 컨트롤러에서 한다.
- app/Builders
    - 복잡한 조건의 객체들을 생성하기 위한 `Builder Pattern`의 클래스들이 있는 곳이다. 예를들어 BEST.GG 내의 LolMatch 클래스는 매우 다양한 조건으로 엄청나게 많은곳에서 각각 다른 조건으로 사용되고 있다. 매번 쿼리를 생성하지 않아도 되도록, 여기서 그를 관리하는 빌더를 생성 할 수 있다.
    - 해당 객체들에 Static Method 를 만들어서 이 기능을 구현할 수 있지만, (Factory Method Pattern) 그 조건들이 복잡해지고 파라미터가 많아질 경우 다음과 같은 문제가 생긴다.
    
        ```
        factory pattern과 abstract factory pattern에는 3가지 중대한 문제점이 있다.
        수 많은 파라메터들이 클라이언트 클래스로 부터 전달 되는데 이것은 에러를 발생시키는 경우가 많다. 왜냐하면 거의 대부분의 경우 argument(인자)들의 type이 같고 클라이언트 쪽에서는 인자들을 정확히 유지시키기 어렵기 때문이다.
        몇몇의 파라메터들은 값을 보낼때 선택적인데 factory pattern에서는 모든 인자를 전송해야 한다. 그리고 선택적인 파라메터들은 꼭 null을 입력해서 보내야한다.
        만약 생성시키는 object가 무거운 경우(파라메터가 많은 경우) 만들기가 매우 복잡해지며, 이런 factory pattern의 복잡성으로 인해 결국 혼란스럽게 된다.
        우리는 이런 문제점을 생성자(constructor)의 인자 갯수를 바꿈으로 해결할 수 있다. 하지만 이런 방식의 문제점은 모든 attribute들이 명시적으로 set 되지 않는 한 object의 상태는 일괄성이 없어지게될 것이다.
        builder pattern은 선택적인 파라메터가 많을 경우 제공 상태를 일관성 있게 해주고, object를 생성시킬때 step-by-step으로 만들 수 있도록 제공해주며 최종에는 만들어진 object를 리턴한다.
        ```
        출처: [seotory님의 블로그](http://seotory.tistory.com/29)
    - 빌더 클래스에는 종종 `Factory Method` 패턴을 적용 할 수 있다.
- app/Services
    - Controller 중, 일부 코드가 너무 길어져서 Service 로직이 별도로 구현 필요한 경우가 있다. Controller 의 구조에 맞게 폴더를 생성 한 후 Service 클래스를 생성하여 조작한다. (Controller 의 구조를 꼭 맞춰야하는건 아니다. 용도에 따라서 다른사람이 보았을 때 잘 알아볼 수 있게만 만들자.)
- app/Models
    - 로직에 필요한 모든 모델들을 포함하고 있다. Trait, Interface, 그리고 각 클래스와 관련 있는 여러 디자인 패턴의 클래스들도 포함한다.
- app/Utils
    - 특정 클래스와 1:1 관련성이 거의 없는 클래스들을 넣어둠. LolPosition 등의 범용적으로 쓰일 수 있는 Trait 도 해당 됨. 예를 들어 아래와 같은 것들일 수 있음
        - Cloudinary: 이미지 프로세싱 관련 Custom Wrapping 라이브러리
        - ConvertKoreanSpeak: 한국어<->영어 발음 전환 라이브러리
        - LolPositionInterface, LolPositionTrait: LolPosition 값이 존재할 경우 사용 할 수 있는 여러가지 메소드 포함 (포지션 정렬, 포지션을 스트링화 시키는 메소드 등)
- tests/API
    - PHPUnit 을 통한 테스트 로직들을 넣어두는 폴더이다. 새로운 API는 항상 테스트케이스를 작성한다. 적어도 일주일 한번은 테스트 케이스를 전부 돌려보도록 한다.
- tests/Models
    - 클래스들의 테스트케이스를 작성한다. 예를 들자면 아래와 같은 테스트 케이스를 작성 할 수 있다.
        ```php
        $champion = \App\Models\Lol\LolChampion::get(1);
        $this->assertTrue($champion->data->getSpells() instanceof App\Models\Lol\LolChampionSpellCollection);
        ```
- 이외 구조들은 Laravel 표준을 따라갑니다. 문서에서 Structure 부분을 필독하길 바람.

# Conventions

## Code Quality

### 코드 삭제를 두려워하지 말자.
여기서 말하는 코드는 `메소드`, `변수`, `주석`, `로깅`, `상수`등 모든 코딩을 말한다. 이 작업은 정말 중요하다. 레거시 시스템을 더욱 레거시하게 만들지 않기 위해서 꼭 신경써야될 부분이다. 동일한 기능을 새롭게 개발하거나 개선할 때, 더이상 사용하지 않는 과거 코드를 **혹시나** 하는 마음에 삭제하지 않고 남겨두는 경우가 있다. 이는 좋지 않다. 그래서,

- 필요 없어졌거나, 더이상 사용하지 않는 코드(메소드, API, Routes 등을 모두 포함)는 항상 삭제한다. 삭제가 두려우면 통주석이나 `throw new Exception` 으로라도 꼭 처리를 한다.

특정 기능을 삭제하거나 변경할 때, 그 기능을 더 이상 사용하지 않는다는 것은 삭제하거나 변경한 자 본인이 제일 잘 알 수 있으니 본인이 하자.

#### 코드들을 이렇게 항상 정리 함으로써 이런 장점이 있다.
- 리팩토링이 습관화 된다. 사용하지 않는 코드나 중복 코드를 삭제하는 것은 리팩토링의 기초이다.
- 제3자 혹은 미래의 내가,
  - 코드를 전체적으로 리팩토링할 시간을 가질 때 사용하지 않는 코드까지 리팩토링 해야될 필요성을 줄일 수 있다. 결과적으로 작업 시간이 줄게된다.
  - 무언가 개발할 때, 애꿎은 (혹은 사용하지 않는) 코드를 건드려야될 확률이 줄어든다. 동일한 코드가 2곳 이상에 존재하면 문제를 느낄 것이고 아래와 같은 해결책이 나올 것이다.
    1. [최악] 시간이 없어서 리서치를 적당히 해보고 적당히 하나만 골라서 수정하게 됨. "하지만 알고보니 전부다 사용중인 코드였다!" (코드 분석 실패, 개발 누락 발생)
    2. [일반적] 어디를 수정해야될지 몰라서 그냥 전부 다 수정을 하게 됨. (안전하지만 추가시간 소요)
    3. [베스트] 어떤 코드가 레거시 코드인지 리서치 후 사용중인 코드만 수정하고 레거시는 삭제하게 됨. (깔끔하지만 처음부터 코드 분석 하는 상황이므로 더 많은 시간 소요) == `이 작업이 바로 여기서 강조하고 있는 '코드 정리'.`
- 오토컴플리트 기능이 매우 강력한 PHPStorm 을 사용하더라도, 특히 Laravel Template 에서는 상대적으로 .php 파일을 다룰 때 보다는 그 기능이 미약하다. 다시 말해 리팩토링 기능이나 Find Usages 를 강력하게 쓰기가 힘들다. 더이상 사용하지 않는 메소드(또는 변수)인지 아닌지 구분하기가 나중에는 쉽지 않다는 것이다.
- 숨어있는 버그를 찾고, 예상치 못한 결과를 방어하기 위한 제일 효율적인 방법이다. (또한, `Unit Tests` 를 돌려보는 것이 매우 큰 도움이 될 수 있습니다.)

### 하드코딩과 땜빵을 하지마세요.
프로젝트의 규모가 어느정도 커지고, 유지보수 과정에 접어들면 흔히 개발자들이,
- "구조적으로 이 기능은 만들 수 없어요."
- "이거 하려면 프로젝트 첨부터 다시 개발해야되요. 리뉴얼 한번 하죠"
- "아 좀 많이 뜯어 고쳐야겠는데.."

와 같은 말들을 슬슬 꺼내기 시작하게된다. (찔림)

사실 당연하게도, 개발중에 기획은 언제나 바뀔 수 있고 실제로 엄청나게 바뀐다. 그로 인해서 코드들은 더러워지게 되고 결국 위와 같은 상황은 당연히 일어날 수 밖에 없다. 하지만 이런 상황을 최대한 늦게, 덜 발생하도록 하는 방법도 없진 않다. 아래와 같이 평소에 대비를 하면 어느정도 위와 같은 말들을 습관적으로 내뱉는 것을 방지할 수 있을 것이다.

- 상황에 따라 다를 수는 있으나, 어쩔 수 없는 상황을 제외하고는 기본적으로 하드코딩과 땜빵을 최대한 지양하도록 한다. 개발 일정이 살짝 지체 될 수 있다고 하더라도, 이는 추후 일어날 수 있는 예상치 못한 많은 문제들, 그리고 개선사항들에 대해 미리 대비하기 위해 매우 중요한 작업이다.
- 위와 같이 미래 대비를 위함도 있지만, 기본적으로 하드코딩과 땜빵을 습관적으로 하는 것은 '프로그래머'가 아닌 '코더'가 될 수 있는 지름길이다.
- 하드코딩을 해야할 상황도 있긴 하다. 하지만, 그런 상황에서는 혼자서 고민하지말고 옆의 백엔드 개발자 동료와 상의/토론하여 최종 합의안을 도출하여 작업하도록 한다. 하드코딩을 하더라도 그나마 덜 허접한 하드코딩을 하자는 것이다.

### PHP Native Array 를 Dictionary 로써 사용하는 것을 지양
PHP 에는 Array 라는 개념이 존재하는데, 이는 Dictionary 와 ArrayList 를 합한 개념이다. 이 두가지 개념이 합쳐진 PHP Native Array 는 Dictionary 로써 사용했을 때, 다방면으로 예측하기가 힘든 자료형이다. 전통적으로 OOP 가 적극적으로 홍보되지 않았던 PHP 인 탓인지, 아니면 매우 사용이 편해서 ($array['key'] 형태로 사용 가능. 심지어 PHP 5.5 버전까지는 `undefined array key` 워닝도 뱉지 않았다.) 그런지 몰라도 PHP 에서는 Native Array 를 남용하는 경우가 적지 않다. 하지만 이것은 아래의 안좋은 상황들을 만든다.
- 무슨 값들이 포함되어 있는지 예측하기가 힘들다. 코드가 5-6줄이면 상관 없다. 하지만 Array 를 계속해서 편하게 쓰다보면 언젠가 그 `$array` 는 저 멀리 있다. 다른 함수에서도 건너가서 쓰고 있다. 파일이 여러개가 되고 있다.
- 어떤 데이터가 포함되어 있는지 따라가기가 너무 힘들어지고 개발중에 매번 `print_r`로 무슨 데이터가 들어있나 체크하면서 개발해야한다. 하지만 "만약 그 배열의 key 가 다이나믹하면?" (아래처럼) 그 배열을 만드는 코드를 자세히 분석해보지 않았었다면 치명적인 오류가 발생할 것이다.
    ```php
    /**
      * @return array
      */
    public function getAmazingData($key) {
        switch($key) {
            case 'a':
                return [
                    'eventType' => 'HOME',
                    'userId' => 2
                ];
                break;

            default:
                return [
                    'eventType' => 'KILL',
                    'killerId' => 7,
                    'victimId' => 3
                ];
        }
    }

    // getAmazingData 함수를 만든 본인이 아닌이상, 처음 이 코드를 봤을 때 getAmazingData 함수를 뜯어보거나 print_r 로 매번 테스트 해보지 않는한 데이터를 '절대' 추측 조차 불가능하다. 추측을 하더라도 정확하지 않을 것.
    $data = getAmazingData($somethingInheritKey);
    ```
- PHPDoc 마저도 Array 안의 데이터에 대한 자동완성은 지원하지 않는다. 즉, 오토컴플리트를 할 수 있는 여지조차 없다.

그래서 Native Array 는 지양하고 최대한 객체를 생성하여 데이터를 관리하도록 하자.

## Strategy

### 네이밍/오타
- 네이밍중 생기는 오타는 철저히 관리한다. 오타는 무조건 제거해주세요.
- 메소드명은 너무 짧게 지으려고 노력하지 않아도 된다. 길더라도 의미가 분명하게 짓도록 한다. (너무 짧으면 리팩토링이 필요할 때 작업이 힘든 점도 존재)
- 메소드의 역할이 바뀌는 경우 메소드명도 필히 수정을 해야한다. 메소드명과 동작이 다르게 되어서는 안된다. 누군가가 놓친 것을 발견하면 역시 수정해주자.

### 자동완성
자동완성은 아래의 두가지 문제를 해결하기 위해서 최대한 할 수 있는 만큼 구축해야한다. 심지어 그것을 위해 메소드를 한개 더 생성하는 한이 있어도 말이다.
- 각 클래스에 어떤 변수나 메소드가 존재하는지 알기 힘든 것을 해결해준다.
- 시도때도 없이 메소드명이나 변수명을 바꾸고 싶을 때 (리팩토링) 자동완성을 (PHPDoc, Type Hinting, ...) 잘 구축해두면 PHPStorm의 `Refactoring` 기능으로 매우 편하게 리팩토링 할 수 있다.

구축 방법은 여러가지가 있지만 대표적으로 아래와 같다.
- 모델이 늘어날 수록 각 클래스에 어떤 변수가 존재하는지 모를 경우가 훨씬 더 늘어난다. ide-helper generator 로 해결 할 수 없는 경우라면, 클래스 상단에 프로퍼티로 `phpdoc` 을 입력해두어 자동완성이 먹히게끔 작업한다.
    ```php
    /**
     * Class LolMatchSetPlayerStat
     * @package App\Models\Draft\Game\lol
     *
     * @property float kda
     * @property float total_cs_per_min
     * @property float damage_dealt_share_rate
     * @property float kill_participation_rate
     * @property int   pp
     */
    class LolMatchSetPlayerStat extends Model
    ```
- 컬렉션이지만 어떤걸 담고 있는 컬렉션이라고 표현하고 싶을 땐, 아래와 같이 `|` 를 써서 `MatchPlayerStat[]` 등으로 표현한다. 좀 더 명확한 설명을 위함이고, 주석의 확장판이라고 보면 된다. (이 phpdoc 은 사실 PHP 네이티브 `foreach` 문을 쓸 때 활용이 된다. 하지만 우리는 ->each(function(`MapPlayerStat` $stat){ }) 나 ->map 을 쓰니까 필요 없다. 그래서 사실 `@return Collection` 만 지정을 해도 문제는 없다. 이에 대한 내용은 아래 'foreach 보다는 Collection 의 each 활용' 섹션에서 자세히 설명한다.)
    ```php
    /** @var MatchPlayerStat[]|Collection $players */
    $players = $myTeam->players->where('isRequester', false);
    ```
    ```php
    /**
     * @return Collection|SummonerPlayedWith[]
     */
    public function getPlayedWith(): Collection
    ```
- 변수들도 아래와 같이 최대한 자동완성을 구축한다. 
    ```php
    /** @var LolPlayer **/
    $player = $players->first();
    ```

## 메소드와 클래스

### self vs static
Static Method 를 호출 할 때, `self::` `static::` 중 특별한 이유가 없다면 static 으로 호출하도록 한다.

> `self` refers to the same class in which the `new` keyword is actually written.

> `static`, in PHP 5.3's late static bindings, refers to whatever class in the hierarchy you called the method on.

From: http://stackoverflow.com/questions/5197300/new-self-vs-new-static

### 메소드 파라미터
메소드 파라미터는 특별한 이유가 있지 않는 한 객체로 한다. 강제는 아니다. (PHP의 타입힌트를 사용하는데 제약이 있다면 phpdoc 을 이용한다.)
```php
public function isWinMatchTeam(LolMatchTeam $lolMatchTeam = null)
{
    if ($lolMatchTeam === null) {
        throw new \Exception("lolMatchTeam 을 null 로 보내다니? 죽을래?");
    }

    return $this->win_match_team_id == $lolMatchTeam->id;
}
```

하지만, 아래와 같이 Controller 에서 어쩔 수 없이 (예를 들자면 `eagerLoad`) ID 값으로만 호출해야 하는 경우에는 ID 값을 파라미터로 받도록 허용한다.
```php
// 아래와 같이 만들면 matchTeamId 에 대한 eagerLoad 가 불가능하기 때문에, isWinMatchTeamId 메소드에서 파라미터를 int id 로 주는 것을 허용한다.
'participants_teams'      => $lolMatchSet->lolMatchSetPlayerStats->groupBy('match_team_id')
    ->map(function (Collection $lolMatchSetPlayerStats, $matchTeamId) use ($lolMatchSet) {
        return [
            'is_win'  => $lolMatchSet->isWinMatchTeamId($matchTeamId)
        ];
    })->values()
```
```php
public function isWinMatchTeamId(int $matchTeamId)
{
    return $this->win_match_team_id === $matchTeamId;
}
```

### foreach 보다는 Collection 의 each 활용
```php
foreach ($lolMatches as $lolMatch) {
    echo $lolMatch->id;
    dd($lolMatch->lolMatchTeams); // 자동완성 안됨
}
```
위 코드는 아래와 같이 대체해서 쓰자. `map` 을 비롯해 비슷한 메소드가 매우 많으니 Laravel 공식 문서의 `Collection`을 참고.
```php
$lolMatches->each(function(LolMatch $lolMatch) use ($myMatchTeam) {
    echo $lolMatch->id;
    echo ($lolMatchTeam->lolMatchTeams[0]->id === $myMatchTeam->id);
    dd($lolMatch->lolMatchTeams); // 자동완성 됨
});
```
- 프로젝트 전체에 걸쳐 php native array 사용은 `절대 지양`하고 `Illuminate\Support\Collection` 사용. (Object 가 필요한 경우 `CustomModel` 생성)
- `array` 대신 `Collection` 을 기본 배열의 단위로 사용하는 Laravel 에서는, `map`와 `each` 메소드를 통해 type-hint 를 이용하여 자동완성을 쉽게 구현 할 수 있다. 이를 활용하지 않고 php 의 foreach 문을 사용하는것은 기껏 phpdoc 과 ide-helper 로 열심히 자동완성을 구축해둔것이 무용지물이 될 수 있다.
- `map` 과 `each` 함수는 익명함수를 응용한 메소드로써 익명함수 내부에는 새로운 `scope` 영역이 생성되기 때문에, 필요한 변수들을 `use` 로 끌어와야한다. 이는 자칫 귀찮고 복잡하다고 생각할 수 있지만 익명함수 내의 코드량이 늘어날 수록 예상치 못한 오류와 코드간의 혼동, 변수간의 혼동을 효과적으로 줄여줄 수 있는 예방책이 될 수 있다.

###### 참고: `eloquent`의 `scope`와 여기서 말하는 `scope 영역`은 다릅니다. `block` 이라고 부르기도 한다.

## DB, Eloquent, Model

### 날짜/시간 필드 생성 네이밍 컨벤션
- 기본적으로 `did_at` 의 형식으로 `TIMESTAMP` 타입의 필드를 생성한다. 그리고 타임존은 무조건 `UTC` 기준으로 한다.
- 만약 아래와 같은 경우 `DATE` 타입을 활용하여 날짜 값만 저장하도록 하며 타임존은 `UTC`가 아닌 로컬 타임존으로 한다. 네이밍 컨벤션은 별도로 정해진바는 없으며 필드명을 `date` 로 사용한 곳도 있다.
  - 타임존을 알 수 없고,
  - 시간 값이 없이 날짜 값만 있으며,
  - 특성상 단순 출력 용도의 느낌이 강한 데이터의 경우

### 메소드에서 `get` 워드의 사용
- `Laravel`의 `QueryBuilder` 객체로 응답이 구성되어지는 메소드는 `get` 을 붙이지 않는다. (사용하려면 `->get()` 을 해야하니까.)
    ```php
    class Test {
        public static function item($id) {
            return LolItem::whereId($id);
        }
    }
    
    // 사용
    dd(Test::item(11)->first(), Test::item(11)->get()->count());
    ```
- 반대로 데이터 자체를 바로 리턴 하는 메소드들은 get 을 붙인다.
    ```php
    class Test {
        public function getTest($id) {
            return LolItem::whereId($id)->first();
        }
    }
    
    // 사용
    dd(Test::getItem(11));
    ```
- 또 아래의 예처럼 될 수 있다.
    ```php
    public function getTitle()
    {
        $team1 = $this->lolMatchTeams->get(0);
        $team2 = $this->lolMatchTeams->get(1);
        return "{$team1->getLolTeam()->slug} vs {$team2->getLolTeam()->slug}";
    }

    public function getOpponentLolMatchTeam()
    {
        return $this->lolMatch->lolMatchTeams->reject(function ($lolMatchTeam) {
            return ($this->competition_team_id == $lolMatchTeam->competition_team_id);
        })->first();
    }
    
    public function getMatchLosses(LolTeam $lolTeam)
    {
        return $this->count() - $this->getMatchWins($lolTeam);
    }
    ```

### Scope 활용
- 특정 테이블에서 새로운 가공 데이터를 뽑을 때, 주요 필드(relation 이 걸려있는 필드들, `player_id`, `match_id`, `match_set_id`, ...)들이 포함되어야 하는 데이터 모델이라면 해당 모델 클래스에서 `Eloquent`의 `scope`를 이용해 쿼리를 만들 수 있도록 노력한다.
- 이는 아래와 같은 상황에서 파라미터 수를 효과적으로 줄일 수 있는 방법이 될 수 있다.
    ```php
    // 선언
    class LolMatchSetPlayerStat extends Model {
        // 메소드 이름이 get 으로 시작하니, 코드 말미에 eloquent->get() 을 포함해야한다.
        public static function getSelectTopPlayedChampionStats(LolPlayer $lolPlayer, $limit, LolCompetition $competition = null, LolTeam $opponentLolTeam = null)
        {
            // ...
        }
        // ...
    }
    
    // 사용
    dd(LolMatchSetPlayerStat::getSelectTopPlayedChampionStats($lolPlayer, 10, $lolCompetition, $opponentLolTeam));
    ```
    아래의 코드로 개선이 가능하다. 결과는 동일.
    ```php
    // 선언
    class LolMatchSetPlayerStat extends Model {
        public function scopeSelectTopPlayedChampionStats(Builder $query, LolCompetition $competition = null, LolTeam $opponentLolTeam = null)
        {
            // ...
        }
        // ...
    }
    
    // 사용
    dd($lolPlayer->lolMatchSetPlayerStats()->selectTopPlayedChampionStats($lolCompetition, $opponentLolTeam)->limit(10)->get());
    ```

### Global Scope 지양
- 프로젝트 규모에 따라서 Global Scope 는 매우 유용한 기능이 될 수도 있다. 하지만 프로젝트와 데이터 규모가 커지고, 데이터 마이닝이 고도화 됨에 따라 Global Scope 기능을 활용하는건 매우 큰 독이 될 수 있다.
- 아래는 잘못된 `GlobalScope` 의 예이다. 실제로 본 프로젝트에서는 `LolMatch`, `LolMatchSet` 클래스를 사용하는 곳이 수백곳이 되는데, 아래와 같은 글로벌 스코프가 걸려져있었고 데이터 마이닝이 고도화되고 데이터 수집 소스들이 늘어남에 따라 `withoutGlobalScope` 를 써야되는 상황이 끝이 없이 찾아왔다. 그래서 이 Global Scope 를 제거하는 대형 작업을 진행한 적이 있다. 이 작업은 객관적으로 매우 힘들고 외로운 작업이며, 누구도 하기 싫어하는 작업임을 공감하지 않는 사람은 아무도 없을 것이다. 예방하자.
    ```php
    class LolMatch extends Model {
        public static function boot()
        {
            return static::addGlobalScope(function (Builder $builder) {
                $tableSelf = self::table();
                
                // 항상 종료된 매치만 가져옴
                return $builder->where("{$tableSelf}.is_resolved", true);
            });
        }
    }
    ```
여담: 프로젝트 내에 Join을 할 때 컬럼들이 마구잡이로 할당되버리는 현상으로 인해 아직 `->select("{$tbSelf}.*")`  를 GlobalScope 로 활용하는 곳은 많다. 지금까지 문제점을 찾진 못했으나 언젠가 독이 될 수도 있을 것 같다는 생각을 하고 있다. (by kars, 2017.04.27)

### Model Attribute 활용
- Laravel Eloquent Model 에서는 `Attribute` 기능(Getter, Setter)을 통해서, 변수를 접근할 때 별도의 Getter 함수를 만듦으로써 여러가지 전처리를 자유롭게 할 수 있다. 하지만 남용하면 정작 오리지널 데이터가 필요할 때 귀찮아지고 예측불가능해지므로, Attribute 는 "형 변환"에 한해서만 쓰도록 정한다. (protected $casts 기능의 대안)
- *특히 이것이 중요한데*, Attribute 를 작성하여 null 체크를 한 뒤 기본값을 내려주는 등의 행위는 절대 하지 않도록 한다. 나중에 해당 객체에 특정 값을 할당해주고 `->save()` 를 할 때 문제가 생기게된다. 동일한 클래스의 다른 객체에서 값을 카피해오던지, 혹은 관리자 페이지를 구축 할 떄라던지 말이다.

### Raw Query 작성시 절대 주의 (보안)
`selectRaw`, `whereRaw`, `orderByRaw`, `DB::raw`, ...
- 위와 같은 메소드등을 활용하여 Raw Query 를 작성 할 때는, 꼭 모든 변수들을 `addslashes` 로 묶는 것을 절대 잊지 않아야한다. 매우 중요하다. 해킹 방지의 목적이다. 제일 간단하지만, 제일 효과적으로 진행할 수 있는 최고의 유일한 예방책이다. (**예외: 변수명이 '쿼리'임을 직설적으로 표현하는 경우에만 예외 - `$queryGroupBy`, `$pppsDataQuery`, ...**)
- 실제 개발중에는 어떤 여러가지 합리적인 이유에 의해 모든 변수들을 꼭 묶을 필요가 없을 것이다. (바로 위에 선언한 값이라서 절대 해킹 될 이유가 없다거나..) *하지만,* 누군가의 그 코드들의 중간 코드 수정으로 인해 상황이 바뀔 수 있고, (그 사람이 앞뒤 코드를 제대로 분석하지 않을 수도 있다.) 그런 경우 심지어 실수로 addslashes 를 씌우는 것을 까먹는 불상사가 생길 수도 있다. 해킹은 한번 발생하면 걷잡을 수 없는 사태이므로 확실히 예방하기 위해 `그냥 뭐든 무조건 전부 addslashes 를 씌우는 것`으로 한다.
- 개발중 빠진 쿼리가 보인다면 적극적으로 고치고 커밋하자. 발견했으면서 귀찮아서 넘어가면, 보고 그냥 넘어간 것 자체도 본인 책임이다.
- 미리 위에서 addslashes 하지말고, 무조건 Query 문을 작성할 때 addslashes 를 씌운다고 생각하면 2중 addslashes 를 한다던지, 안한다던지 하는 실수를 하지 않을 것이다. 옛날 옛적 2000년대 초중반에 작성된 PHP 코드들의 DB를 열어보면 `\'`, `\"`, `\\` 등의 역슬래쉬 데이터가 2중, 3중으로 들어가있는 것들을 보았는가? 이런 햇갈리는 코딩 규칙 때문이었다. 아직도 PDO 를 사용하지 않는 레거시 코드 작성자가 있다면 이 상황은 지금도 없어지지 않아을 것이다.
- 이로 인한 사고 발생시 웹서버 상단에 WAF 설치 여부를 떠나, 그 책임은 개발자의 몫이 될 수 있다는 점을 잊지 말자. 
    ```php
    $tbSet        = LolMatchSet::table();
    $tbPlayerStat = LolMatchSetPlayerStat::table();
    $tbMatch      = LolMatch::table();
    
    $expression   = LolMatchSetPlayerStat::getPPSelectExpression();
    $rName        = 'posRank';
    $dataSetQuery = "(
                SELECT  
                    player_id,
                    pp, 
                    total_sets,
                    @" . addslashes($rName) . ":=@" . addslashes($rName) . " + 1 ranks
                FROM    
                    ((SELECT 
                        player_id,
                        count(*) as total_sets,
                        avg(" . addslashes($expression) . ") as pp
                    FROM `" . addslashes($tbPlayerStat) . "` AS sub_stats
                    JOIN `" . addslashes($tbSet) . "` sub_lol_match_sets ON sub_lol_match_sets.id = sub_stats.match_set_id
                    JOIN `" . addslashes($tbMatch) . "` sub_lol_matches ON sub_lol_matches.id = sub_lol_match_sets.match_id
                    WHERE sub_lol_matches.competition_id = " . addslashes($competition->id) . " AND sub_stats.position = '" . addslashes($position) . "'
                    GROUP BY player_id
                    HAVING total_sets >= 10)) AS sub,
                    
                    (SELECT @" . addslashes($rName) . ":=0) sr                            
                ORDER BY pp DESC
            )
        ";
    ```

## Custom Model/Collection
- 모델이 필요하지만 DB 연결은 필요없는 경우 `CustomModel` 과 `CustomCollection` 을 사용한다.
- `Eloquent` 의 `Model`, `Collection`과 유사한 클래스이다. 기본적인 `캐스팅`, `getter`, `setter` 등을 지원한다. 매우 비슷한 방식으로 동작한다.

### CustomModel 의 기본 구조
```php
/**
 * Class MatchPlayerStat
 * @package App\Utils\OpggAPI\Models
 *
 * @property string              position                    "MID"
 * @property string              championId                  103
 * @property int[]|Collection    items                       array:5 [▶]
 * @property Player[]|Collection relatedPlayers              array:5
 * @property SummonerProfile     summonerProfile             array:5
 * @property float               killShareRate               0.44
 * @property integer             keystoneMasteryId           6362
 */
class MatchPlayerStat extends CustomModel
{
    protected $casts = [
        'kill'   => 'int',
        'death'  => 'int',
        'assist' => 'int'
    ];
    
    /** @var Match */
    public $match = null;
    
    public function getSummonerProfileAttribute($value)
    {
        return new SummonerProfile($value);
    }
```
```php
$a = new MatchPlayerStat([
    'position' => 'MID',
    ...
]);
```
- 초기화에서 사용할 CustomModel에 들어갈 변수들은 모두 phpdoc 으로 클래스 상단에 선언한다. 자동완성을 위해서이다.
- 초기화시(`new Object($data)`) 사용될 `$data`는 모두 int, string 등으로 구성된 array 데이터라야만된다. Object 이면 안된다. 마치 MySQL에서 가져온 것 처럼. (Object 가 아니어야하는 이유는 아래에서 설명한다.)
- `$data`가 Nested Array 로 구성된 경우, 위의 `getSummonerProfileAttribute()` 처럼 getter 을 생성하여 `new SummonerProfile($value)` 해서 쓰도록 한다. `$value`는 Array 이다.
- 값들의 캐스팅이 필요하면 `$casts` 를 사용한다. 캐스팅은 Laravel 의 Eloquent 와 호환된다. `jessengers\model\src\Model.php:687` `function castAttribute` 을 확인해보면 좀 더 정확한 지원 목록을 볼 수 있다.

### CustomModel 에서 주의할 사항
- 만약 이미 Object 로 되어 있는 데이터를 해당 커스텀 모델에 넣고 싶으면, 위 예제의 `public $match = null` 처럼 실제 멤버 변수를 선언해서 직접 주입하여 사용하도록 한다.

    굳이 이렇게 하는 이유는 다음과 같다. 초기화에서 사용할 데이터에 오브젝트도 포함되기 시작하면 Nested 형태로 커스텀 모델을 생성해야 할 경우 통일성을 잊게 된다. 결국, 아래와 같은 '개발자의 깊은 고뇌가 담긴 코드'가 나오게 되버릴 것이다. 모든 커스텀 모델을 관리할 때 아래와 같은 IF문을 반복하게 될 용감한 첫 걸음이 될 것이다.
    ```php
    # Before
    public function getSummonerProfileAttribute($value)
    {
      return new SummonerProfile($value);
    }
    ```
    ```php
    # After
    public function getSummonerProfileAttribute($value)
    {
      if (is_array($value)) {
          return new SummonerProfile($value);
      } elseif ($value instanceof SummonerProfile) {
          return $value;
      } else {
          throw new \Exception("알 수 없는 타입의 변수가 summonerProfile 변수에 들어와있습니다.");
      }
    }
    ``` 


## Cache

캐시 기능은 매우 효과가 좋은 성능 개선 도구이다. 하지만 그를 위해서 잃어야 하는 것들이 매우 많다. 예를 들자면 엄청나게 무거운 연산을, 캐시 단 한방으로 100배 이상의 성능을 개선할 수 있는 대신에 데이터가 그동안 변화할 수 없으므로 실시간 데이터 업데이트가 필요한 상황에서는 엄청난 독약이 될 수 있다.

그 특성은 개발뿐만 아니라 기획과도 크게 연관된다. 데이터가 업데이트 되어야 하냐 마냐에 대한 판단은 기획적인 판단을 위주로 해야하며, 기술쪽에서는 최대한 효율이 좋은 캐시 시간을 기획에 제안하여 서로 조율 후 TTL 을 정하는게 이상적이다. 이 과정에서 최대한 누구나 인식할 수 있는 쉬운 규칙을 짜서 어떤 것들은 30분, 어떤 것들 3시간, 어떤 것들은 24시간, 어떤 것들은 7일등으로 3~5종류 정도로만 나눠서 관리를 하여야 관리하기가 쉽고, 기획자가 `캐시`에 대한 부분을 잠시 잊거나 혼동하였을 때 '이거 왜 업데이트 안되요?'라는 질문에 대해 쉽게 그 답안을 떠올리고 설명해줄 수 있게 해준다. 

### 쿼리 캐시를 위해 [rememberable](https://github.com/dwightwatson/rememberable) 라이브러리를 사용한다.
쿼리 캐시를 위해 이 라이브러리를 사용중이다. 자세한 사용법은 [Github](https://github.com/dwightwatson/rememberable) 에서 확인하시기 바랍니다. 이 함수의 사용 특성상 relation 에서 활용하고 싶을 수 있습니다. 그럴 경우 아래에 정리된 문제들이 생길 확률이 매우 크므로 절대 삼가기 바란다.

### Cache 기능은 최대한 제약적으로, 최대한 껍데기 레벨에서 사용한다.
- `rememberable`의 경우, Eloquent Model 의 relation 에서는 절대 사용하지 않는다.
- 특별한 이유가 없는 한, Eloquent Model 이나 Custom Model 의 `getSomething` 형의 메소드에서는 Cache 를 최대한 사용하지 않는다. 이 외에도 `Model`에서 사용하는 경우 자체를 최대한 지양하자. 남이 물어봤을 때 그 이유를 설명하고 동의하게끔 설득 시킬 수 있을 만큼만 사용하도록하자. (제일 흔한 것은 스태틱 메소드나 `Factory Method Pattern`이 될 것이다.)
- 모든 Cache 기능은 최대한 `Controller`, `Service` 패턴에서만 한하여 쓰도록 한다. 

이로써 아래의 이슈에 대한 원천 해결을 할 수 있다.
- 소스를 처음 접하거나 남이 만든 모델을 처음 사용할 때, 그리고 그 모델의 특정 메소드들을 사용해서 원하는 데이터를 추출 할 때, 데이터가 계속해서 변경이되지 않거나 나오지 않는 현상이 생길 수 있다. 이 경우 캐시에 대한 충분한 학습이나 인지가 없다면 이 현상이 나오는 원인을 추측하기가 매우 힘들고 불가능에 가깝다.
- `rememberable` 라이브러리는 `$queryBuilder->remember()` 라는 말도 안되는 쉬운 사용성으로 편함을 제공하지만, 익숙하지 않는 자에게는 그만큼 추측하기 힘든 동작을 제공하기도 한다. 난 릴레이션에 접근해서 데이터를 가져왔을 뿐인데, 데이터가 자꾸 반영이 안된다고 생각 해보라.

### TTL 은 해당 클래스의 상단에 const 로 선언한다.
- `Controller`, `Service` 등에서 `Cache` 클래스나 `->remember()`을 사용할 때, 그 TTL 을 해당 영역에 하드코딩 하지 않도록 한다. TTL 은 클래스 상단에 아래와 같이 선언하도록 한다. 필요할 때 마다 `Cache::remember(, 1, )` 등으로 직접 숫자를 넣어서 사용하면 나중에 햇갈릴 확률이 너무 크다.

    ```php
    class DummyController {
        const CACHE_TTL_MINUTE_SHIT = 60; // 똥쌀 때 60분 캐시
        
        // ...
        function myCode() {
            $a = SomeModel::wherName('a')->remember(static::CACHE_TTL_MINUTE_SHIT);
            ...
        }
    }
    ```
