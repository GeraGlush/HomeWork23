using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Wheat : MonoBehaviour
{
    private int wheat;
    private int workers;
    private int soliders;

    public Player player_cs;

    private void Start()
    {
        ChangeSoliders();
        ChangeWheat(3);
    }

    public int GetWheat()
    {
        return wheat;
    }

    public void ChangeWheat(int toChange)
    {
        wheat += toChange;
        player_cs.wheatText.text = "Wheats: " + wheat;
    }


    public int GetSoliders()
    {
        return soliders;
    }

    public void ChangeSoliders()
    {
        if (!CheckWheat(3))
        {
            return;
        }

        soliders += 1;

        SetText(player_cs.solidersText, "Soliders:", soliders);
        SetText(player_cs.wheatText, "Wheat:", wheat);
    }

    public void SetSolidersByWar()
    {
        soliders -= Random.Range(0, player_cs.EnemySoliderBySurge[player_cs.surge] - 1);

        ///You need at
        /// least 1 soldier to count your winnings
        ///

        SetText(player_cs.solidersText, "Soliders:", soliders);
    }


    public int GetWorkers()
    {
        return workers;
    }

    public void ChangeWorkers()
    {
        if (!CheckWheat(1))
        {
            return;
        }

        workers += 1;

        SetText(player_cs.workersText, "Workers:", workers);
        SetText(player_cs.wheatText, "Wheat:", wheat);
    }

    private void SetText(Text textToSet, string toSetString, int toSetInt)
    {
        textToSet.text = toSetString + " " + toSetInt;
    }

    private bool CheckWheat(int toCheck)
    {
        if (wheat >= toCheck)
        {
            return true;
        }
        return false;
    }

    public void DestroyWheat(int toDestroy)
    {
        wheat -= toDestroy;
        player_cs.wheatText.text = "Wheats: " + wheat;
    }
}

using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using UnityEngine.SceneManagement;

public class Player : MonoBehaviour
{
    private enum Type
    {
        Solider,
        Worker,
        Wheat
    }


    public Wheat wheat_cs;

    public Text soliderBySurgeText;

    public Text surgeText;
    public int surge;

    public Text wheatText;
    public Text workersText;
    public Text solidersText;

    public Button soliderButton;
    public Button workerButton;

    public Image timerToNextSurge;

    public int[] EnemySoliderBySurge;

    public GameObject losePanel;
    public GameObject winPanel;

    private void Start()
    {
        StartCoroutine(SurgeTimer());
    }

    private void Update()
    {
        CheckButton(soliderButton, 3);
        CheckButton(workerButton, 1);
    }

    private void CheckButton(Button button, int toCheck)
    {
        if (wheat_cs.GetWheat() < toCheck)
        {
            button.interactable = false;
        }
        else
        {
            button.interactable = true;
        }
    }

    private IEnumerator SurgeTimer()
    {
        float timer = 0f;

        while (surge != 20)
        {
            soliderBySurgeText.text = "Solider by surge: " + EnemySoliderBySurge[surge];   
            timer = 1f;

            while (timer > 0.1f)
            {
                yield return new WaitForSeconds(0.1f);
                timer -= 0.001f;
                timerToNextSurge.fillAmount = timer;
            }

            wheat_cs.ChangeWheat((wheat_cs.GetWorkers() * 2));

            if (!CheckWin())
            {
                losePanel.SetActive(true);
                break;
            }

            surge++;
            surgeText.text = "Surge: " + surge;
        }

        winPanel.SetActive(true);
    }

    public bool CheckWin()
    {
        if (EnemySoliderBySurge[surge] == 0)
        {
            return true;
        }

        if (wheat_cs.GetSoliders() > EnemySoliderBySurge[surge])
        {
            wheat_cs.SetSolidersByWar();
            return true;
        }
        return false;
    }

    public void TakeSolider(Image timerTake)
    {
        wheat_cs.DestroyWheat(3);
        StartCoroutine(Timer(timerTake, 1f, Type.Solider));
        StartCoroutine(SetClick(soliderButton));
    }

    public void TakeWorker(Image timerTake)
    {
        StartCoroutine(Timer(timerTake, 1f, Type.Worker));
        wheat_cs.DestroyWheat(1);
    }

    private IEnumerator Timer(Image timer, float TimeToSet, Type type)
    {
        float timerFloat = TimeToSet;

        while (timerFloat > 0f)
        {
            yield return new WaitForSeconds(0.001f);
            timerFloat-=0.001f;
            timer.fillAmount = timerFloat;
        }

        if (type == Type.Solider)
        {
            wheat_cs.ChangeSoliders();
        }

        if (type == Type.Worker)
        {
            wheat_cs.ChangeWorkers();
        }
    }

    public void Restart()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }

    public void SetButtonClick(Button button)
    {
        StartCoroutine(SetClick(button));
    }

    IEnumerator SetClick(Button button)
    {
        button.gameObject.SetActive(false);
        yield return new WaitForSeconds(15f);
        button.gameObject.SetActive(true);
    }
}
